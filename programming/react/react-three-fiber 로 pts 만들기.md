## 0. 결과물 부터
![92157b00-1106-11ec-9595-c4488b6ae434](https://user-images.githubusercontent.com/22016317/149983005-7809ce34-caae-45f9-a783-0de03e2edae2.gif)



- [Threejs](https://threejs.org/) : 웹 브라우저에서 애니메이션 3차원 컴퓨터 그래픽스를 만들고 표시하기 위해 사용되는 크로스 브라우저 자바스크립트 라이브러리
- [React-Three-Fiber](https://github.com/pmndrs/react-three-fiber): `React`에서 `Threejs` 를 선언적, 재사용성 있게 개발할 수 있는 라이브러리. `Threejs` 의 `React` 버전

----------------------

웹 개발자가 그래픽스 작업을 할 일이 많지 않으나, 이번 기회가 있을때 쟁취하여 작업하게 되었습니다.
대학때도 제일 재밌게 공부했었는데 여전히 너무 재밌네요. 조직내 이런 컨텐츠를 또 하게된다면 제가 하고싶습니다.
정리하는 느낌으로 쓰자니 길어질것 같아 그나마 잘 읽히고 사연이 있어 보이게 스토리텔링으로 글을 구성했습니다.

-------------------

## 1. 발단
- 연초 올림픽 전에 스포츠 서비스 중 제일 무겁고 복잡한 컨텐츠인 게임엔드를 react로 전환 한답니다.
- 대부분의 페이지는 걍 api 호출해서 데이터 받고 ui 그리고 event handler 붙이면 끝인데,,, 그래픽스 작업을 하고싶었던 제가 할당받게 됩니다.

## 2. 일단 페이지 구조보단, 적용할 기술부터 파악했어야했습니다.
먼저 일반 `threejs` 와 `react-three-fiber`
정육면체 하나가 `(0, 0, 0)` 에서 뱅글뱅글 돌고있는 동일한 스펙의 화면으로 비교를 해보겠습니다.
![1861dd80-b4d6-11eb-8615-3aebd7a4dc93](https://user-images.githubusercontent.com/22016317/149983060-eebba0a2-24b1-4609-80f1-9d85aafc0043.gif)


- Threejs
```javascript
<script>
	//scene 만들고 카메라 만들기
	const scene = new THREE.Scene();
	const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );

	const renderer = new THREE.WebGLRenderer();
	renderer.setSize( window.innerWidth, window.innerHeight );
	document.body.appendChild( renderer.domElement );

	//cube를 만들때 geometry, material를 선언하고 cube 에 넣은 뒤 scene 추가
	const geometry = new THREE.BoxGeometry();
	const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
	const cube = new THREE.Mesh( geometry, material );
	scene.add( cube );

	camera.position.z = 5;

	const animate = function () {
		requestAnimationFrame( animate );
		//매 프레임 마다, 큐브에게 x축, y축기준 0.01 씩 회전을 줌
		cube.rotation.x += 0.01;
		cube.rotation.y += 0.01;

		renderer.render( scene, camera );
	};

	animate();
</script>
```
- React-Three-Fiber (이하 react 라고 그냥 하겠습니다.)
```javascript
import ReactDOM from 'react-dom'
import React, { useRef, useState } from 'react'
import { Canvas, useFrame } from '@react-three/fiber'

function Box(props) {
  //Box 참조를 위함
  const mesh = useRef() 
  const [hovered, setHover] = useState(false)
  const [active, setActive] = useState(false)
  // 여기가 돌리는 것임. useFrame hook 을 이용
  useFrame((state, delta) => (mesh.current.rotation.x += 0.01))

  return (
    <mesh
      {...props}
      ref={mesh}
      scale={active ? 1.5 : 1}
      onClick={(event) => setActive(!active)}
      onPointerOver={(event) => setHover(true)}
      onPointerOut={(event) => setHover(false)}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color={hovered ? 'hotpink' : 'orange'} />
    </mesh>
  )
}

ReactDOM.render(
  <Canvas>
    <Camera/>
    <ambientLight />
    <pointLight position={[10, 10, 10]} />
    <Box position={[-1.2, 0, 0]} />
  </Canvas>,
  document.getElementById('root'),
)
```

느낌만 보면 아시겠지만, `threejs` 의 경우 뭔가 줄 글을 보는 느낌입니다. 
scene 을 만들고,  geometry(모양) 와 material(색) 을 합치며 Box mesh를 만들어내고, 
원하는 위치에 갖다 놓는걸 소설책 읽듯이 읽어갈수 있습니다. 
다만 문제는 분명히 소스코드 내용이 길어질수록 아무리 코드를 잘 분리하더라도 앞에 것을 까먹기 떄문에... 
분명히 가독성 및 이해도가 떨어질 것입니다.

`react` 에선 마지 unity 게임프로그래밍 하듯이, 
`Canvas` 안에 `카메라`넣고, `빛`넣고 `박스`넣고, 
박스는 내부적으로 `boxGeometry`, `meshStandardMaterial`  가지며 
react 컴포넌트 답게 각각은 상태관리를 합니다. 

--------------------

## 3. 기존의 스펙과 구조를 파악해보자
- 야구를 그닥 좋아하진 않지만 간간히 보긴 했으니, 코드를 파봤습니다.
안타깝게도 화면으로 스펙은 가볍게 파악했으지만 구조나 코드는 이해할 수가 없었습니다. (작업자 blame 은 아닙니다.ㅎㅎ) 
- 빨간색으로 선 그은 파일 제외하고 모두 관여를 하고 있는데, 
각각은 몇 백줄에 산문 소설책처럼 써잇는 상황..!
- 투구 데이터 호출을하고 하단 텍스트와 싱크를 맞추는 등 복잡한 타이밍 로직이 전부 들어가 있어서 매우 난처했습니다.
- 결국 기존코드 구조파악 및 참고를 포기하고, **같은 스펙을 정리해 똑같이만 만들기로 결심합니다.**

## 4. 공 날아오는 것의 구현
- 이 모듈의 핵심이었는데요. 공의 `xyz시작위치` + `xyz축 별 초기속도` + `xyz축 별 가속도` 만 있으면 기본적인 등가속도 운동 공식을 활용하여 원하는 순간 위치를 알아낼 수 있습니다. 
`const displacement = (p: number, v: number, a: number, t: number) =>  p + v * t + (1 / 2) * a * t * t`
`const velocity = (v: number, a: number, t: number) => v + a * t`
- `투구 궤적 = 공 + 궤적` 이므로 아래와 같이 구성했고,, 공의 위치와 속도, 궤적 좌표를 매 프레임 마다 상태 업데이트를 해줍니다.
```javascript
  const startPosition = new THREE.Vector3(pitch.x0, pitch.z0, -pitch.y0);
  const startVelocity = new THREE.Vector3(pitch.vx0, pitch.vz0, -pitch.vy0);
  const startAcceleration = new THREE.Vector3(pitch.ax, pitch.az, -pitch.ay);

  const [trajectory, setTrajectory] = useState<Vector3[]>([]);
  const [position, setPosition] = useState<Vector3>(startPosition);
  const [velocity, setVelocity] = useState<Vector3>(startVelocity);
  // 매 프레임 속도와 위치 궤적 추가
  useFrame((state, delta) => {
    const newPosition = calculatePositionAtTime(position, velocity, acceleration, delta * velocityRatio,);
    const newVelocity = calculateVelocityAtTime(velocity, acceleration, delta * velocityRatio);

    setPosition(newPosition);
    setVelocity(newVelocity);
    setTrajectory([...trajectory, newPosition]);
  });
 return 
 <>
   <Ball
       position={position}
       color={0xffffff}
   />
   <BallTrajectory
     trajectories={_.uniqWith(trajectory, _.isEqual)}
     color={colorMap.trace}
   />
 </>
```
- 공 상태중 위치만 업데이트 하는것이기 떄문에 rerender 할 때 공의 위치만 정확하게 변화하여 그려질 것입니다.

![image](https://user-images.githubusercontent.com/22016317/149983520-f6cb47d5-b553-451c-a526-42d734696b0f.png)

## 5. 공을 여러개 그리기
- Api 데이터는 3개가 한번에 내려오는데 공 3개를 동시에 발사하는게 아닌 **순차적으로 공을 발사하는 기능이 필요**했습니다.
- 스크린야구장 가면 있는 `PitchingMachine` 컨셉을 이용했습니다.  `PitchingMachine` 은 실제 화면에 그릴 `renderingQueue`와 `waitingQueue` 를 배열로 가지고 있고, Api로 받아온 데이터는 하나씩 `waitingQueue` 에 집어넣습니다.  그리고 timer 로 1초마다 `waitingQueue`에서 pop 한 뒤 `renderingQueue` 로 밀어넣기로합니다. 
```javascript
const PitchingMachine = ({fireAllOneTime, pitchingDelay,}: PitchingMachineProps) => {
  const [waitingQueue, setWaitingQueue] = useRecoilState(PitchingMachine.waitingQueue);
  const renderingQueue = useRecoilValue(PitchingMachine.renderingQueue);
  
  const moveOneByOneToRenderingQueue = useRecoilCallback(
    PitchingMachine.moveOneByOneToRenderingQueue,
  );
  useEffect(() => {
   // 공 밀어넣는 과정.
    const pitchUnits: PitchUnit[] = [];
    for (const i in batter.textOptions) {
      const value = batter.textOptions[i];
      pitchUnits.push({ textOption: value, ptsOption: pts });
    }
    setWaitingQueue(pitchUnits);
  }, [batter.ptsOptions.length, batter.no]);

  // delay 마다 한번씩 공 발사해줌.
  useInterval(() => {
       const first = moveOneByOneToRenderingQueue();
  }, pitchingDelay);

  // reneringQueue 에 있는걸 꺼내서 그리자~
  return (
    <>
      {renderingQueue.map((value) => (
        <Pitch
          key={value?.ptsOption?.pitchId}
          pitch={value?.ptsOption}
          colorMap={PtsDrawConst.BALL_COLOR_MAP(value?.textOption?.pitchResult)} //볼이면 초록 스트라이크면 노랑 같은식..
        />
      ))}
    </>
  );
};
```
![e1f93f80-1110-11ec-986b-04e6de7906fb](https://user-images.githubusercontent.com/22016317/149982891-6a42dd26-07c7-4903-9914-e175a94180ff.gif)



## 6. 카메라 자유도를 제거하고 ㅠㅠ 마크업 붙이기
- 3d로 카메라 돌리면서 공 궤적을 자유롭게 볼 수 있도록 개발을 했지만..... 
- 추가 인터렉티브 조직 쪽 리소스 불가! 및 운영 스펙에 의해 카메라는 포수 시점으로 고정이되고, 이전 투구도 볼 수 없게 됩니다.
- 캔버스 위로 선수 마크업 + 경기장 마크업, 이닝정보 마크업 붙이는 노가다를 실시하여 결과물을 완성합니다.
- 데이터 fetch 하는 모듈은 따로 선언하여 가져온 pts 정보는 `PitchingMachine`의 상태만 업데이트 해주고, 
-  문자 중계데이터 정보는 하단에 별개의 모듈로 데이터를 내려주고,
- `PitchingMachine` 은 자신이 들고있는 공 그리기에 충실합니다.
- 서비스 스펙 맞추느라 코드는 조금 지저분해졌지만 역할은 명확하게 나뉘어있습니다.

![image](https://user-images.githubusercontent.com/22016317/149983195-c659a743-476c-4d34-b19e-fed1758f7a64.png)

## 7. 성능
- mesh 의 조각이 많아질수록 화면 그리는게 힘듭니다. 
특히 반응형으로 만드는 만큼, webgl 을 소화할 수 있으나 사양이 떨어지는 폰은 브라우저가 멈추는 상황도 있었습니다.
- 궤적을 그리는데 사용되는 CatmullRomCurve3 가 성능 저하의 원인이었는데요, 
조금 각져보이더라도, 좀 덜그려지게해 성능에 문제없도록 했습니다. (QA분과 적정값 조정)

## 8. 후기
- 다 작업하고 나니 threejs 결과물 보다 react로 그리도록 작업한 것이 코드양 및 가독성에 훨씬 좋았습니다. 
그냥 제가 느끼기엔요.ㅎㅎ 
![image](https://user-images.githubusercontent.com/22016317/149983242-8fae4ed7-40f7-4004-a034-3b1105864ce2.png)
- react 는 `상태 관리`를 이용해서 화면을 그리는 녀석이기 떄문에, 스펙이 상태관리에 매우 의존적으로(?) 되는 느낌입니다. 
단순화를 하려면 Api의 데이터가 어떤 상태로 오면 뭔가 데이터를 킵하거나 딜레이할 수 있는 것이 아닌 무조건! 그대로 그리기는 방식이기  때문에, 미처 소화하지 못한 기존 스펙도 있긴 합니다.
