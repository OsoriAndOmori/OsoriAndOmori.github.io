---
title: Spring Batch Chunk Oriented & Transaction
author: OsoriAndOmori
date: 2022-07-15 18:00:00 +0900
categories: [Blogging, Spring]
tags: [spring, spring-batch, transaction]
---

## Spring Batch Chunk Oriented & Transaction
- 맨날 StepBuilder.java 나 JobBuilder.java 로 기계적으로 job 을 만들다 보면, 너무나 많이 추상화 된 프레임워크에 의해 가축이 되는 느낌을 받습니다...

--------------

- Spring Batch 는 chunk 단위로 데이터 처리를 합니다.
    - Job - Step - Tasklet - Reader + Writer + Process 계층을 이루고 있는 점.
    - 그리고 Tasklet 내 chunk 단위 처리 중 에러가 발생하면 롤백 한다는 내용은 누구나 알고 있습니다.
- 일단 jobLauncher 가 Job 을 시작하는건 알겠는데 어떻게 R + P + W 순으로 동작을 하는걸까 (이하 RPW)
- 그리고 분명히 spring-tx 의 TransactionManager을 이용할거고, 트랜잭션 처리를 할 것 인데 어느 타이밍에 트랜잭션이 동작하는지 모르겠다는 의문이 들었습니다.
    - writer 에 쓰기전에 transaction 을 실시할까? 그러면 processor 에서 뭔가 insert 한건 롤백이 안될 것 같은데...
    - 그럼 read 할 때부터 transaction 을 거는건가? 뭣하러 그때부터 걸지?
    - writer 를 Composite Writer 를 써서 멀티 database 에 writer 를 하는 경우는 무엇을 해야할까
    - 등등의 의문을 가졌습니다.
- 이걸 확인하기 위해선 크게 두 줄기로 파봤어야합니다.

---------------

#### 1. Job 은 어떻게 만들어지는가 - Chunk Oriented 구현 방식에 관한 이야기
- 뭔가 writer 의 write() 를 호출하기 전후로 트랜잭션 코드가 있을것 같은 느낌을 받았습니다.
- 그래서 Job, Step 구성하는 소스를 까봐야했죠.
```java
@Configuration
@RequiredArgsConstructor
public class JobConfig {
    private final JobBuilderFactory jobBuilderFactory;
    private final StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job sampleJob(Step sampleStep, JobExecutionListener jobExecutionListener) {
        return jobBuilderFactory.get(JOB_NAME)
                .start(sampleStep)
                .listener(jobExecutionListener)
                .preventRestart()
                .build();
    }

    @Bean
    @JobScope
    public Step sampleStep(ItemReader<BeforeDomain> reader,
                           ItemProcessor<BeforeDomain, AfterDomain> processor,
                           CompositeItemWriter<AfterDomain> writer) {
        return stepBuilderFactory.get(JOB_NAME.concat("Step"))
                .<BeforeDomain, AfterDomain>chunk(CHUNK_SIZE)
                .reader(reader)//user_reward
                .processor(processor)
                .writer(writer)
                .build();
    }
}
```
- 흔한 Job 설정이나 Job 을 만들 떄 쓰는 `JobBuilderFactory`, `StepBuilderFactory` 를 보겠습니다.
- 이 Bean 들은 `@EnableBatchProcessing` 활성화 시, [SimpleBatchConfiguration.java](https://docs.spring.io/spring-batch/docs/current/api/org/springframework/batch/core/configuration/annotation/SimpleBatchConfiguration.html) 를 통해 자동으로 생성됩니다.
- StepBuilderFactory.java 를 살펴보면, transactionManager 를 주입받아놓고, get 할 때 같이 넣어줘서 만드는 것을 알 수 있습니다. 원한다면 `.transactionManager()` 를 통해 기본말고 커스텀하게 집어넣을 수 있죠.
```java
    public StepBuilderFactory(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
		this.jobRepository = jobRepository;
		this.transactionManager = transactionManager;
	}

	public StepBuilder get(String name) {
		StepBuilder builder = new StepBuilder(name).repository(jobRepository).transactionManager(
				transactionManager);
		return builder;
	}
```
- StepBuilder 가 transactionManager 를 넣어준다는건 알았으니, 뭔가 트랜잭션 관리를 잘 해줄거라는건 보이긴하고,
- 그럼 RPW 는 실제로 어떻게 구성이 되는지 알아보기 위해선 조금 더 StepBuilder 를 파봐야합니다.
- StepBuilder.build() 에선, 어느 순간 `createTasklet()` 이라는걸 호출하게됩니다. 결국 RPW 도 하나의 Tasklet 으로 구성이 되는거고, Step 은 Tasklet 을 품고 있는 우리가 아는 구조가 되나봅니다.
```java
	@Override
	protected Tasklet createTasklet() {
        ...
		SimpleChunkProvider<I> chunkProvider = new SimpleChunkProvider<>(getReader(), repeatOperations);
		SimpleChunkProcessor<I, O> chunkProcessor = new SimpleChunkProcessor<>(getProcessor(), getWriter());
		ChunkOrientedTasklet<I> tasklet = new ChunkOrientedTasklet<>(chunkProvider, chunkProcessor);
        ...
		return tasklet;
	}
```
- 다 생략하고 chunkProvider 에 reader 를 넣고, chunkProcessor 에 processor + writer 를 집어넣어 `ChunkOrientedTasklet.java` 를 만들게 됩니다.
- 결국 이 녀석이 핵심인것 같습니다. 내부 execute 코드를 살펴봐야했는데 chunkProvider 의 실제 구현체 SimpleChunkProvider.java 를 보면 우리가 아는 Reader 와 크게 다르지 않습니다.
- 그런데 SimpleChunkProcessor 는 특이합니다.
```java
    //ChunkOrientedTasklet.java
	@Override
	public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
        ...
        // 처리해야할 데이터
		Chunk<I> inputs = (Chunk<I>) chunkContext.getAttribute(INPUTS_KEY);
		if (inputs == null) {
			inputs = chunkProvider.provide(contribution); //더이상 인풋이 없을때까지 될 때까지 계속 읽는 영역
			if (buffering) {
				chunkContext.setAttribute(INPUTS_KEY, inputs);
			}
		}

        //SimpleChunkProcessor
		chunkProcessor.process(contribution, inputs); //process + writer
		chunkProvider.postProcess(contribution, inputs);
        ...
	}

    //SimpleChunkProcessor.java
    @Override
    public final void process(StepContribution contribution, Chunk<I> inputs) throws Exception {
        initializeUserData(inputs);
        if (isComplete(inputs)) {
            return;
        }

        // Processor 가 지지고볶음
        Chunk<O> outputs = transform(contribution, inputs);

        contribution.incrementFilterCount(getFilterCount(inputs, outputs));

        // Wirter 가 지지고볶음
        write(contribution, inputs, getAdjustedOutputs(inputs, outputs));
    }
```
- 대충 정리해보면 chunk 단위로 읽을 떄서 있습니다.
- [spring batch reference](https://docs.spring.io/spring-batch/docs/current/reference/html/step.html#chunkOrientedProcessing) 에 가니 정확하게 알려주네요.
![img](https://user-images.githubusercontent.com/22016317/178551961-accebb1c-1ece-4662-b66f-8a004fc3b8a9.png)
- 이로써 spring batch 는 ChunkOrientedTasklet.java 를 유연하게 만들어서 RPW 를 주입받고, 이를 통해 chunk 지향처리를 진행함을 알 수 있었습니다.
- 하지만 정작 궁금했던 Transaction 이 걸리는 타이밍에 관해서는 더 들어가도 위 코드 분석으로는 알 수 없었습니다. 어디서 거는건지 알 수가 없네요.

#### 2. Job 은 어떻게 실행되는가 - 트랜잭션에 관한 이야기
- Transaction Manager 를 stepBuilder 에서 주입했는데, chunk 단위 처리를 하는 Tasklet 상 코드를 볼 수 없었습니다.
- 제가 처음 예상했던 그림은 아래와 같았는데 여지없이 빗나가서 Job 실행단계를 파봐야함을 느꼈습니다.
```java
  //ChunkOrientedTasklet.java 내
  read()
  트랜잭션매니저.doTransaction start
  process()
  writer()
  트랜잭션매니져.commit()
```
- Job을 실행할 때 `JobLauncher.launch()` 를 통해 실행을 합니다.
- 계속 메서드를 따라 들어가다보면, TaskletStep -> doExecute 에서 아래를 볼 수 있습니다.
```java
new TransactionTemplate(transactionManager, transactionAttribute)
  .execute(new ChunkTransactionCallback(chunkContext, semaphore));
```
- Step 의 ChunkContext를 spring-tx 의 TransactionTemplate 위에서 동작 시키는 겁니다.
- 요걸 Callback 형태로 실행을 해서 따라가기가 좀 힘들긴한데, 결국 step 의 transactionManager 위에서 Chunk Tasklet 자체가 실행됨을 볼 수 있습니다.
- 결론은 나왔습니다. read 할 때부터 transaction 이 걸리는거고, 중간 processor에서 insert 를 하건 뭘하건, 어디든 한 곳에서 뻑나면 모두 rollback이 될 수 있다는걸 소스를 통해 확인했습니다.
```java
   //TaskletStep.doInTransaction : 트랜잭션 안에서 이 일을해라...
    @Override
    public RepeatStatus doInTransaction(TransactionStatus status) {
        TransactionSynchronizationManager.registerSynchronization(this);
        ...
        result = tasklet.execute(contribution, chunkContext); //여기!
        ...
      }
```


## 후기
- 왜 제 예상대로 만들지 않고, 실행단계에서 Chunk 단위로 처리할때 복잡한 콜백 패턴으로 만든건지 생각을 해봣는데, 결과적으로 이게 더 분업에 효율적이고 모듈화를 잘 된 케이스라고 보입니다.
- 왜냐면 `1번 Job 을 어떻게 구성하는가` 영역은 말그대로 Job 을 "구성" 만의 개발만하면 되는 것이고,
- `2번 Job은 어떻게 실행되는가` 를 구현하는 이들은 "실행"단 개발에 집중할 수 있기 떄문이 아니었을가 싶네요.
- 결국 스프링도 여러 개발자들의 협업에 의해 개발 되므로, Runner 단에서 트랜잭션 처리 개발과 순수 Job 구성단의 분업을 깔끔하게 하려고해서 그러지 않았나? 생각이 듭니다.
- 이건 시간날때 한번 batch repo 에 질문해볼 수 있을 것 같네요.
- 긴 글 읽어주셔서 감사합니다.
