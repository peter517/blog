title: Unity中自定义VR事件输入模块
tags: [Unity,VR]
date: 2016-12-10 10:30:50
description: 怎么在Unity中自定义一个类似鼠标的输入模块
---

PC平台可以用鼠标作为移动输入模块，移动平台上面有touch模块，Unity上面作为跨平台游戏引擎，也都支持这些事件，但是在VR领域中，鼠标、touch这些输入模块都是跟业务场景无法联系，所以Unity支持开发者自定义输入模块，以一个UI组件模拟鼠标来发出输入事件

# 主要思路
在VR领域，主要可以获取到的用户信息就是陀螺仪，程序可以默认在屏幕中央构建一个输入点，当陀螺仪变化的时候，输入点也会随之而变，当输入点和某些UI重合时，就能出发点击等事件

# 实现方式
继承BaseInputModule模块，Process方法会不断的被调用，在这里面来定义PointerEventData事件，通过获取mouse的位置来创建RaycastAll事件，同时通过ExecuteEvents.Execute<IPointerClickHandler>来发送点击事件
## 定义模块
```
public class MyInputModule : BaseInputModule {

	private PointerEventData pointerData;
	public Transform mouse;

	/// @cond
	public override bool ShouldActivateModule() {
		bool activeState = base.ShouldActivateModule();
		return activeState;
	}
	/// @endcond

	public override void DeactivateModule() {
		eventSystem.SetSelectedGameObject(null, GetBaseEventData());
	}

	public override bool IsPointerOverGameObject(int pointerId) {
		return pointerData != null && pointerData.pointerEnter != null;
	}

	public override void Process() {
		if (pointerData == null) {
			pointerData = new PointerEventData(eventSystem);
		}

		// Cast a ray into the scene
		pointerData.Reset();
		pointerData.position = Camera.main.WorldToScreenPoint (mouse.position);
		eventSystem.RaycastAll(pointerData, m_RaycastResultCache);

		pointerData.pointerCurrentRaycast = FindFirstRaycast(m_RaycastResultCache);
		m_RaycastResultCache.Clear();

		HandlePointerExitAndEnter(pointerData, pointerData.pointerCurrentRaycast.gameObject);

	}

	void Update(){
		if (Input.GetKeyDown (KeyCode.Space))
		{
			if(pointerData != null && pointerData.pointerCurrentRaycast.gameObject != null)
				ExecuteEvents.Execute<IPointerClickHandler>(pointerData.pointerCurrentRaycast.gameObject, pointerData,
					ExecuteEvents.pointerClickHandler);
		}
	}
}

```

## UI获取事件
UI组件挂上了这个脚本后就能获取自定义输入模块发出的事件
```
public class PointerClickHandlerListener : MonoBehaviour,IPointerClickHandler, IPointerEnterHandler,IPointerExitHandler {

	public void OnPointerClick(PointerEventData eventData)
	{
		Debug.Log("OnPointerClick " + this.name);
		if (eventData.pointerId == -1)
			Debug.Log("Left Mouse Clicked");
		if (eventData.pointerId == -2)
			Debug.Log("Right Mouse Clicked");
	}

	public void OnPointerEnter(PointerEventData eventData)
	{
		Debug.Log("Pointer Enter " + this.name);
	}

	public void OnPointerExit(PointerEventData eventData)
	{
		Debug.Log("Pointer Exit");
	}
}
```

<font color="#FF0000">版权声明：本文为博主原创文章，转载请注明出处</font>
