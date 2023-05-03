# 以时间状态做划分

当前状态、前置状态、下一个状态、状态变化条件



# 状态

可以是一个或者多个动作组合在一起

好处：无论多么复杂的行为都能在一个状态中体现出来



# 程序实现

## 状态基础类：AIStateBase  

​	父类构建状态

扩展状态机：子类继承衍生

​	重写 **OnEnter、OnExit、Update**

**AI 逻辑都放在了每个状态里**



```c#
class AIStateBase
{
    private int STATE;
    public abstract OnEnter();
    public abstract OnExit();
    public abstract Update();
    
}

//跑步状态
class AIRunState: AIStateBase
{
    public AIRunState()
    {
        STATE = STATE.AIRunState;
    }
    
    public override OnEnter()
    {
        //播放动作
    }
    public override OnExit()
    {
     	//停止动作   
    }
    public override Update()
    {
        //如果有目标 就向目标移动
        //如果没有目标 就向指定方向移动
    }
}

//追击状态：包含两个动作 追+击
class AIMoveAttackState : AIStateBase
{
    public AIMoveAttackState()
    {
        STATE = STATE.AIMoveAttackState;
	}
    
    public void SetTarget(Actor target)
    {
        currentTarget = target;
	}
    
    public override OnEnter()
    {
        //播放动作
    }
    public override OnExit()
    {
     	//停止动作   
    }
    public override Update()
    {
        //检查条件
        //if(..)
        //在监视范围内进行追击

        //在攻击范围内进行攻击
        Attack();
        
        //目标超出监视范围外，进入停止攻击状态


    }
    
    private void Attack()
    {
        //可以根配置数据 做动画和攻击时间点的变化
        //可以根据技能编辑器技能数据在某个点执行某个动作或者释放某个特效
    }
    
}



```



## 状态控制器：控制状态

状态管理类，对状态实例管理和转换的作用，没有太多的逻辑



作用：

1、存储所有状态

2、切换状态

3、记录当前的状态

4、降低逻辑耦合



```c#
class StateControl
{
    AIStateBase[] mStates;
    AIStateBase mCurrentState;
    public void Init()
    {
        //初始化
        mStates = new AIStateBase[STATE.Max];
        
        //.. 装载
        mStates[STATE.AIRunSatet] = new AIRunState(StateControl);
        mStates[STATE.MoveAttack] = new AIMoveAttackState(StateControl);
        
        mCurrentState = null;
        
    }
    
    public void ChangeState(STATE state)
    {
        //切换状态
        if(mCurrentState != null)
        {
            mCurrentState.OnExit();
        }
        mCurrentState = mStates[state];
        mCurrentState.OnEnter();
    }
    
    void Update()
    {
        //实时更新
        if(mCurrentState != null)
        {
			mCurrentState.Update();
        }
	}
}
```





# 状态的三个必要的接口

Update：更新函数

OnEnter：进入状态

OnExit：离开状态



# 进阶：技能系统或者Action系统

一系列可编辑、可调试的动作集合

不需要编写代码

对时间线和节点进行编辑



# 缺点

不能处理和模仿太复杂的智能行为

突发情况会让状态的数量指数级攀升



# 解决：行为树
