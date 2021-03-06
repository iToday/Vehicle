# 设计初衷
车机中包含多种状态，在不同状态下，系统的功能表现不同；

如：

蓝牙电话时，需要打开显示屏和触摸屏，媒体静音，暂停播放，屏蔽部分功能按键等；

倒车时，需要打开显示屏，关闭触摸，媒体静音，暂停播放，屏蔽所有按键等；

待机时，需要打开显示屏，关闭触摸，屏蔽部分功能按键等；

需要设计一种机制统一管理状态的切换及行为，并且各状态下的行为可通过配置文件配置；

# 车载状态机
设计车载状态机，解决复杂的状态问题

下面是状态机核心部分的类图
![](https://github.com/iToday/Vehicle/blob/master/doc/Overview%20of%20ivi.jpg)

我们将车机中状态分原子状态和组合状态；原子状态不可分割，组合状态由原子状态组成；</br>
原子状态包括静音开关状态、触摸屏开关状态、显示屏开关状态；</br>
组合状态包括电话状态、倒车状态、声控状态、待机状态等；如电话状态时，显示屏打开、触摸打开、声音打开；倒车状态时，显示屏打开、触摸关闭、声音关闭；</br>

`原子状态`
```Java

public  class AtomicState extends Object{
	/**
	 * 状态优先级
	 */
	public static final int PRIORITY_HIGH = 0;
	
	public static final int PRIORITY_NORMAL = 1;
	
	public static final int PRIORITY_LOW = 2;
	
	public static final int PRIORITY_LOWEST = 3;
	
	
	/**
	 * 状态值
	 */
	public static final int ON = 1;
	
	public static final int OFF = 0;
	
	/**
	 * 状态类型
	 */
	public static final String TYPE_MUTE = "mute";
	
	public static final String TYPE_SCREEN = "screen";
	
	public static final String TYPE_TOUCH = "touch";
	
	/**
	 * 返回值
	 */
	public static final int SUCCESS = 0;
	
	public static final int FAILED = -1;
	
	private int priority = PRIORITY_LOWEST;
	
	private int state = OFF;
	
	//from will be phone / backcar / standby etc.   
	private String from = null;
	
	private String type = null;
	
	public AtomicState(int state, int prority, String type,  String from){
		this.state = state;
		this.priority = prority;
		this.type = type;
		this.from = from;
	}
	
	/**
	 * 比较两种状态的优先级
	 * @param dest
	 * @return true ：大于等于目标状态优先级，否则 false
	 */
	public boolean comparePriority(final AtomicState dest){
		return dest != null && priority >= dest.priority;
	}
	
	/**
	 * 比较两种状态是否相同
	 * @param dest
	 * @return true:与目标状态相同，否则 false
	 */
	
	@Override
	public boolean equals(Object o) {
		if (o == null || !(o instanceof AtomicState))
			return false;
		
		AtomicState dest = (AtomicState) o;
		
		return  priority == dest.priority 
				&& state == dest.state
				&& type.equals(dest.type)
				&& from.equals(dest.from);
	}

	@Override
	public int hashCode() {
		return toString().hashCode();
	}

	/**
	 * 获取当前状态
	 * @return
	 */
	public int getState(){
		return state;
	}
	
	public boolean equalsState(final AtomicState dest){
		return dest != null && dest.getState() == state;
	}
	
	/**
	 * 判断当前状态是否from相同
	 * @param from
	 * @return true 相同，否则 false
	 */
	public boolean isFrom(String from){
		return this.from != null && this.from.equals(from);
	}
	
	/**
	 * 判断当前状态是否type相同
	 * @param type
	 * @return true 相同，否则 false
	 */
	public boolean isType(String type){
		return this.type.equals(type);
	}

	@Override
	public String toString() {
		return getClass().getSimpleName() + " [priority=" + priority + ", state=" + state
				+ ", from=" + from + ", type=" + type + "]";
	}
	
}
```
`组合状态`
```Java
/**
 * 复合状态，如电话/倒车/待机等
 * 复合状态包括一个或多个AtomicState
 * @author itoday
 *
 */
public class CompositeState extends Object {
	
	/**
	 * 按键优先级
	 */
	public static final int PRIORITY_HIGH = 0;
	
	public static final int PRIORITY_NORMAL = 1;
	
	public static final int PRIORITY_LOW = 2;
	
	public static final int PRIORITY_LOWEST = 3;
	
	public static final int MSG_START_ACTIVITY = 1;
	
	public static final int MSG_SCREEN_DISPLAY = 2;
	
	public static final int MSG_SRC_MODE = 3;
	
	public static final int MSG_MEDIA_MODE = 4;
	
	public static final int MSG_SEND_BROADCAST = 5;
	
	protected  ArrayList<AtomicState> astates = new ArrayList<AtomicState>();
	
	private int keyPriority;
	
	private Handler mHandle;
	
	/**
	 * 构造函数
	 * @param from
	 * @param keyPriority
	 */
	public CompositeState( Handler handle, int keyPriority){
		this.keyPriority = keyPriority;
		mHandle = handle;
	}
	
	/**
	 * 获取原子状态
	 * @param type
	 * @return
	 */
	public AtomicState getAtomicStateByType(String type){
		
		for (AtomicState state : astates) {
			if (state.isType(type))
				return state;
		}
		
		return null;
	}
	
	/**
	 * 处理按键
	 * @param key
	 * @param action
	 * @param step 
	 * @return true 表示按键已处理，false 表示按键未处理
	 */
	public boolean onKey(int key, int action, int step){
		return false;
	}
	
	/**
	 * 比较按键优先级
	 * @param dest
	 * @return 优先级大于返回true
	 */
	public boolean comparePriority(CompositeState dest){
		return dest != null && keyPriority < dest.keyPriority;
	}
	
	@Override
	public boolean equals(Object o) {
		
		if (o == null || !(o instanceof CompositeState))
			return false;
		
		CompositeState dest = (CompositeState) o;
		
		if (this.keyPriority == dest.keyPriority 
				&& astates.size() == dest.astates.size()){
			for (int index = 0; index < astates.size(); index ++) {

				if (!astates.get(index).equals(dest.astates.get(index)))
					return false;
			}
			
			return true;
		}
		
		return false;
	}

	@Override
	public String toString() {
		return getClass().getSimpleName() + " [astates=" + astates + ", keyPriority=" 	+ keyPriority + "] \n";
	}
	
	@Override
	public int hashCode() {
		return toString().hashCode();
	}
		
	public void startActivityWithAction(String action){
		
		Message msg = mHandle.obtainMessage(CompositeState.MSG_START_ACTIVITY);
		msg.obj = action;
		mHandle.sendMessage(msg);
	}
	
	public void setScreen(boolean on){
		
		Message msg = mHandle.obtainMessage(CompositeState.MSG_SCREEN_DISPLAY);
		msg.arg1 = on ? 1 : 0;
		mHandle.sendMessage(msg);
	}
	
	public void startSrcMode(){
		
		Message msg = mHandle.obtainMessage(CompositeState.MSG_SRC_MODE);
		mHandle.sendMessage(msg);
	}
	
	public void startMediaMode(){
		
		Message msg = mHandle.obtainMessage(CompositeState.MSG_MEDIA_MODE);
		mHandle.sendMessage(msg);
	}
	
	public void sendBroadcast(String action, String key, int value){
		
		Message msg = mHandle.obtainMessage(CompositeState.MSG_SEND_BROADCAST);
		
		Bundle bundle = new Bundle();
		bundle.putString("action", action);
		bundle.putString("key", key);
		bundle.putInt("value", value);
		
		msg.setData(bundle);
		
		mHandle.sendMessage(msg);
	}
	
}
```
`状态生效`
```Java
//
private void update() {
  AtomicState asMute;
  AtomicState asScreen;
  AtomicState asTouch;

  synchronized (cstates) {
    asMute = getTargetState(AtomicState.TYPE_MUTE);
    asTouch = getTargetState(AtomicState.TYPE_TOUCH);
    asScreen = getTargetState(AtomicState.TYPE_SCREEN);
  }

  if (DEBUG)
    Log.d(TAG, "thread run: mute :" + asMute + "   \n last mute: " + lastMute);

  if (asMute != null && !asMute.equalsState(lastMute)){
    //设置静音状态,所有声音静音，调用硬件静音
    mPlatform.setMute(asMute.getState() == AtomicState.ON);
    lastMute = asMute;
  }

  if (DEBUG)
    Log.d(TAG, "thread run: screen :" + asScreen + "   \n last screen: " + lastScreen);

  if (asScreen != null &&  !asScreen.equalsState(lastScreen)){

    //设置屏幕开关
    mPlatform.setBacklight(asScreen.getState());
    lastScreen = asScreen;
  }

  if (asTouch != null && !asTouch.equalsState(lastTouch)){

    if (DEBUG)
      Log.d(TAG, "thread run: touch :" + asTouch);

    //设置触摸状态
    mPlatform.setTouch(asTouch.getState());
    lastTouch = asTouch;
  }
}
```
