#iOS 9 Day by Day
#12. GameplayKit - Behaviors & Goals

10번 포스트에서 우리는 지정된 장애물들을 피하며 씬(Scene)의 두 지점 사이의 길을 계산하기 위해 GameplayKit의 pathfinding API를 어떻게 사용할 수 있는지 살펴보았다.

이 포스트는 우리는 씬을 통과해 노드를 움직이는 다른 접근 방식을 취할 것이다. Gameplay kit은 행동(Behaviours)와 목표(Goals)이라는 개념을 소개했다. 그것을 우리에게 제약(constraints)과 원하는 성취(achievements)를 기반으로 씬의 노드들 배치하는 방법을 제공한다. 더 자세히 살펴보기 전에 이것이 어떻게 작동하는지 예제를 살펴보자.

<video width="100%" height="500" controls loop>
	<source src="images/Day12_Missile.mov" type="video/mp4">
	Your browser does not support the video tag.
</video>

우리가 곧 만들어 볼 위의 예제에서는 우리는 사용자를 나타내는 노란색 상자를 볼 수 있다. 이 상자는 씬 주위에 움직이는 사용자의 손가락으로 조종된다. 매우 기초적인 것이다! 흥미로운 부분은 플레이어를 탐색(Seek)하는 미사일이다. 이 미사일은 언제나 플레이어 노드의 중심 포인트에 도달하려고 할 것이다.

이것은 물리나 특화된 코드 어떤 것도 사용하지 않고, 오직 하나의 탐색 목표로 이루어지는 간단한 행동에 의해 조종된다.

이제 우리는 행동과 목표의 일에 대해 조금 알고 있다. 이 데모 앱을 어떻게 만들지 살펴보자.

##Behavior and Goal 예제 만들기

이 예제를 어떻게 만드는지 같이 살펴보자.

![기본 SpriteKit 템플릿 설정하기](images/Day12_setup.png)

기본 SpriteKit 템플릿을 설정하고 GameScene.swift 파일을 연다.

처음으로 해야 할 일은 우리의 엔티티들을 설정하는 것이다.

```swift
let player:Player = Player()
var missile:Missile?
```

GKEntity는 기능성을 제공하는 컴포넌트를 추가할 수 있는 범용 객체이다. 우리의 경우 플레이어를 나타내는 것과 미사일을 나타내는 다른 하나 두 개의 엔티티를 가지고 있다. 우리는 이것들을 어떻게 설정하는지 조금 더 자세히 살펴볼 것이다.

우리는 이 엔티티들뿐 아니라 컴포넌트 시스템의 배열을 설정해야 한다. 컴포넌트 시스템은 같은 타입으로 호출될 컴포넌트들의 동질(homogeneous)의 컬렉션이다. 우리는 여기서 lazy var 프로퍼티를 사용할 수 있다. 왜냐하면 우리는 처음 컴포넌트 시스템이 사용 될 때 한번 초기화되는 것을 원하기 때문이다. 우리는 타겟팅을 위한 컴포넌트를 가지고 있다. (플레이어 노드를 겨냥할 수 있도록 미사일 노드에 추가된다) 그리고 렌터링을 위한 컴포넌트도 가지고 있다. (그래서 씬의 두 엔티티를 렌더링할 수 있다) 우리가 반환하는 컴포넌트의 순서대로 컴포넌트가 실행될 것이다. 그래서 우리는 타켓팅 컴포넌트 다음 렌더링 컴포넌트를 반환할 것이다. 왜냐하면 우리는 노드의 위치가 타겟팅 컴포넌트에 의해 업데이트된 다음 화면에 그려지기를 원하기 때문이다.

```swift
lazy var componentSystems:[GKComponentSystem] = {
	let targetingSystem = GKComponentSystem(componentClass: TargetingComponent.self)
	let renderSystem = GKComponentSystem(componentClass: RenderComponent.self)
	return [targetingSystem, renderSystem]
}()
```

그런데 GameKit 컴포넌트는 무엇인가? 우리는 그것이 씬의 엔티티들에 주는 영향에 대해 논의했지만 그것이 실제로 무엇을 하는지 논의하지 않았다. GKComponent는 엔티티 안 객체의 특정 부분에 대한 데이터와 로직을 캡슐화한다. 컴포넌트는 엔티티와 연관되어 있지만, 엔티티들은 여러 컴포넌트가 있을 수 있다. 컴포넌트들은 엔티티에 추가될 수 있는 재활용 가능한 행동의 조각을 제공한다. 컴포넌트는 구성 패턴(composition pattern)을 사용하여 큰 규모의 게임에서 문제가 될 수 있는 거대한 상속 트리(inheritance tree)를 예방하는데 도움이 된다.

이 씬의 두 엔티티 모두 렌더링 컴포넌트를 갖고, 추가로 미사일 엔티티는 타겟팅 컴포넌트를 가지고 있다.

###엔티티들 설정하기

#### 플레이어(Player) 엔티티

다음은 플레이어 클래스이다. 이는 오직 하나의 컴포넌트를 가지고 있는 NodeEntity의 간단한 서브 클래스로 또한 `GKAgent2D` `agent` 프로퍼티를 가지고 있다.

`GKAgent2D`는 차례대로 `GKComponent`의 서브클래스인 `GKAgent`의 서브클래스이다. `GKAgent`는 자신의 지역 좌표계(local coordinate system)가 속도(Velocity)에 따라 조정되는 질량을 가진 점이다. `GKAgent2D`는 `GKAgent`의 2차원에 특화된 클래스이다.

```swift
class Player: NodeEntity, GKAgentDelegate {
	let agent:GKAgent2D = GKAgent2D()
```

이 경우, 에이전트는 바보(dumb)이다. 실제로는 사용자 조작에 의해 수동으로 노드 위치를 변경하지 않는 한 아무것도 하지 않거나 노드의 위치에 영향을 주지 않는다. 타게팅 컴포넌트는 타겟으로 사용할 에이전트를 가지고 있었야 하므로 에이전트가 필요하다.

```swift
override init() {
	super.init()
```

init 함수에서 RenderComponent를 추가하고, 렌더 컴포넌트의 노드에 PlayerNode를 추가한다. PlayerNode에 대해선 자세한 설명을 하지 않겠다. 그건 지루하고 단지 노란색 상자를 그릴 뿐이다!

```swift
let renderComponent = RenderComponent(entity: self)
renderComponent.node.addChild(PlayerNode())
addComponent(renderComponent)
```


우리는 또한 에이전트의 delegate를 self로 설정해야 한다. 그리고 실제로 엔티티에 에이전트를 추가한다.

```swift
agent.delegate = self
	addComponent(agent)
}
```

또한 만약 에이전트가 업데이트되면 노드 위치(position)가 업데이트되고, 만약 노드 위치를 수동으로 업데이트하는 경우 에이전트의 계산이 수행되기 전에  에이전트의 위치가 업데이트되도록 GKAgentDelegate 함수를 구현해야 한다.

```swift
	func agentDidUpdate(agent: GKAgent) {
		if let agent2d = agent as? GKAgent2D {
			node.position = CGPoint(x: CGFloat(agent2d.position.x), y: CGFloat(agent2d.position.y))
		}
	}

	func agentWillUpdate(agent: GKAgent) {
		if let agent2d = agent as? GKAgent2D {
			agent2d.position = float2(Float(node.position.x), Float(node.position.y))
		}
	}
}
```


#### 미사일(Missile) 엔티티

미사일 엔티티는 PlayerNode와는 약간 다르다. 생성자에서 우리는 미사일이 탐색할 타겟 에이전트를 전달한다.

```swift
class Missile: NodeEntity, GKAgentDelegate {

	let missileNode = MissileNode()

	required init(withTargetAgent targetAgent:GKAgent2D) {
		super.init()

		let renderComponent = RenderComponent(entity: self)
		renderComponent.node.addChild(missileNode)
		addComponent(renderComponent)

		let targetingComponent = TargetingComponent(withTargetAgent: targetAgent)
		targetingComponent.delegate = self
		addComponent(targetingComponent)
		}
```

당신은 이 클래스에는 바보(dumb) GKAgent2D가 없다는 것을 눈치챘을 것이다. 왜냐하면, 우리는 씬 주위로 엔티티를 움직이기 위해 TargetingComponent를 사용하기 때문이다. 아래에서 TargetingComponent에 대해 다룰 것이다. 지금은 당신이 알아야 할 모든것은 우리가 targetAgent를 생성자에서 타겟팅 컴포넌트로 전달해야 한다는 것이다. 그리고 타겟팅 컴포넌트가 delegate 메서드를 작동(trigger)시킬 것이다.

이것을 위해 우리는 다시 `agentDidUpdate`와 `agentWillUpdate` delegate 메소드를 구현해야 한다. 이 메소드들이 플레이어에 있는 것과 어떻게 다른지 주목하라. 이 경우 두 메소드안에서 zRotation을 고려 해야 한다.

```swift

	func agentDidUpdate(agent: GKAgent) {
		if let agent2d = agent as? GKAgent2D {
	    			node.position = CGPoint(x: CGFloat(agent2d.position.x), y: CGFloat(agent2d.position.y))
	    			node.zRotation = CGFloat(agent2d.rotation)
			}
	}

	func agentWillUpdate(agent: GKAgent) {
		if let agent2d = agent as? GKAgent2D {
    			agent2d.position = float2(Float(node.position.x), Float(node.position.y))
    			agent2d.rotation = Float(node.zRotation)
	}
}
```


### 타겟팅 컴포넌트

지금까지 모든 클래스들은 비교적 가벼웠다. 당신이 우리 게임이 잘 작동하도록 타겟팅 컴포넌트가 논리와 코드로 가득 차 있어야 한다고 생각해도 무리가 아니다. 하지만 다행히도 GameplayKit 덕분에 그렇지 않다! 전체 클래스는 단지 20라인이다.

```swift
class TargetingComponent: GKAgent2D {

	let target:GKAgent2D

	required init(withTargetAgent targetAgent:GKAgent2D) {

		target = targetAgent

		super.init()

		let seek = GKGoal(toSeekAgent: targetAgent)

		self.behavior = GKBehavior(goals: [seek], andWeights: [1])

		self.maxSpeed = 4000
		self.maxAcceleration = 4000
		self.mass = 0.4
	}
}
```

코드는 너무 간단해서 설명할 것이 많지 않다. 당신은 클래스가 `GKAgent2D`의 서브클래스인 것을 알 수 있고, `toSeekAgent`생성자로 `GKGoal`을 만든다. 이 목표는 다음 `GKBehavior` 객체를 생성하는 데 사용된다. 만약 복수의 목표를 가지고 있다면, 예를 들어 특정 타겟을 탐색하지만 다른 것은 피해야 하는 경우, 생성자에 복수의 목표를 전달할 수 있다. 또한 각 목표마다 특정한 가중치를 지정할 수 있다. 만약 하나의 에이전트를 피하는 것이 다른 것을 탐색하는 것보다 중요하다면 여기서 나타낼 수 있다.

또한 아랫부분에서 `maxSpeed`, `maxAcceleration` 와 `mass`의 값을 설정한다. 이 단위들은 차원은 없지만(dimensionless) 관련되어 있다. 이 값들은 당신의 정밀한 시나리오에 따라 달라진다. 나는 올바른 값을 얻기 위해 시간이 걸렸다. 처음에 나는 아무 일도 일어나지 않는 줄 알았는고 어디가 잘못되었는지 찾기 위해 한참을 소비했다. 이 값들이 모두 기본값으로 설정되어 있던 것을 밝혀냈다. 내 미사일 노드는 움직였지만, 정말 정말 느렸다!

### 미사일 노드

이제 미사일 엔티티를 설정했다. 우리는 씬에서 미사일 엔티티를 시각적으로 표현하기 위해 노드를 생성해야 한다. 이 노드는 단지 하나의 함수를 가진 SKNode 서브클래스이다.

```swift
func setupEmitters(withTargetScene scene:SKScene) {
	let smoke = NSKeyedUnarchiver.unarchiveObjectWithFile(NSBundle.mainBundle().pathForResource("MissileSmoke", ofType:"sks")!) as! SKEmitterNode
	smoke.targetNode = scene
	self.addChild(smoke)

	let fire = NSKeyedUnarchiver.unarchiveObjectWithFile(NSBundle.mainBundle().pathForResource("MissileFire", ofType:"sks")!) as! SKEmitterNode
	fire.targetNode = scene
	self.addChild(fire)
}
```

볼 수 있듯이, setupEmitters 함수는 씬 객체를 받고, 두 개의 SKEmitter 노드를 생성한다. 이 이미터들을 미사일 노드 자신에 추가하고 이미터의 타겟 노드로 씬 객체를 설정한다. 만약 타겟 노드를 설정하지 않는다면 방출된 파티클은 단지 미사일과 함께 머물고, 씬에서 안 움직이는 것으로 보인다. 이 두 이미터들은 프로젝트에 .sks 파일로 설정된다. `MissileFire.sks` 와 `MissileSmoke.sks` 원한다면 살펴 보자. 여기에서는 자세히 들어가지 않을 것이다.

### 부품들을 결합하기

이제 우리의 노드, 엔티티와 구성 요소가 모두 설정되었다, `GameScene.swift`로 돌아가서 모두 함께 넣어보자! 우리는 `didMoveToView`를 오버라이드(override) 해야 한다.

```swift
override func didMoveToView(view: SKView) {
	super.didMoveToView(view)
```

초기화 중 이미 플레이어를 설정했다. 그래서 씬에 player.node를 간단히 추가 할 수 있다.

```swift
	self.addChild(player.node)
```

미사일의 경우 우리는 이 메소드에서 설정 해야 한다. 미사일의 타겟으로 플레이어의 에이전트를 설정해야 한다.

```swift
	missile = Missile(withTargetAgent: player.agent)
```

그다음 우리는 또한 앞서 논의한 대로 이미터가 미사일과 같이 움직으는 대신 자취를 남길 수 있도록 setupEmitters 함수에 씬을 전달해야 한다.

```swift
	missile!.setupEmitters(withTargetScene: self)
	self.addChild(missile!.node)
```

마지막으로 두 엔티티가 모두 설정되면, 우리의 컴포넌트 시스템에 엔티티의 컴포넌트들을 추가할 수 있다.

```swift
		for componentSystem in self.componentSystems {
			componentSystem.addComponentWithEntity(player)
			componentSystem.addComponentWithEntity(missile!)
		}
```

이제 `update:currentTime` 함수에서 우리가 해야 할 전부는 componentSystems 배열의 모든 컴포넌트 시스템을 델타 타임으로 업데이트 하는 것이다. 이것은 행동들이 무효화(invalidate)하고 재계산(recalculate)을 한 다음, 렌더링을 트리거거 한다.

```swift
override func update(currentTime: NSTimeInterval) {

	// 마지막 'update'가 실행 된 후 얼마나 시간이 지났는지 계산한다.
	let deltaTime = currentTime - lastUpdateTimeInterval

	for componentSystem in componentSystems {
		componentSystem.updateWithDeltaTime(deltaTime)
	}

	lastUpdateTimeInterval = currentTime
}
```

그리고 그게 전부이다! 이제 게임을 실행하면 플레이어를 향해 질주 미사일을 볼 수 있다. 불행하게도 우리는 충돌 감지와 폭발을 추가하지 않았다. 추가 연습 문제로 폭발 컴포넌트를 직접 만들어 보는 것도 좋다!

##Further Reading
이 포스트에서 다룬 새로운 GameplayKit에 대한 더 많은 정보는 WWDC 세션 608,[Introducing GameplayKit](https://developer.apple.com/videos/wwdc/2015/?id=608)을 찾아봐라. 잊지 말자. 만약 이 포스트에서 만들고 설명한 프로젝트를 한 번 시도해 보려면 [GitHub](https://github.com/shinobicontrols/iOS9-day-by-day/tree/master/12-GameplayKit-Behaviors)에서 찾을 수 있다.

만약 지난 두 개의 GameplayKit 포스트가 재미있었다면 이번 포스트에서 논의한 행동과 목표 API를 사용해 pathfinding을 통합하는 예제를 만들어 보는 것도 좋다.

만약 질문이나 코멘트가 있다면 우리는 피드백을 듣기를 원한다. [@christhegrant](http://twitter.com/christhegrant)로 트윗을 보내거나, [@shinobicontrols](http://twitter.com/shinobicontrols)를 팔로우해서 iOS9 Day-by-Day 시리즈의 최신 뉴스나 업데이트 소식을 얻을 수 있다!
