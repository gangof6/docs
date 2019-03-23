



### 杜绝硬编码

#### 场景1 

通常会把各种状态定义成`enum`，如：
```java
public enum UserStatus{
    STATUS1,
    STATUS2,
    STATUS3,
    STATUS4,
    STATUS5;
}
```

如果需要根据状态来限制某种操作的话，经常有人会这样写：
```java
if(UserStatus.STATUS1.equals(status)
    || UserStatus.STATUS2.equals(status)){
    //do something
}
```
这是一种怀味道。

试想，如果这种逻辑很多个地方都需要的话，上面这种代码会散落在很多文件中。当`UserStatus`中增加一个或者减少一个状态的时候，就需要改很多地方。这是一件很恐怖的事情。更恐怖的是，当增加一个状态机，却没有把这个新状态有关的逻辑添加到诸多代码判断条件中时，程序会出现逻辑上的错误！

**所有有关状态的判断都应该是收敛的，至少收敛到一个文件中**

**优化1**

```java
public enum UserStatus{
    STATUS1,
    STATUS2,
    STATUS3,
    STATUS4,
    STATUS5;
    
    public static boolean canDoSomthing(status){
        if(UserStatus.STATUS1.equals(status)
            || UserStatus.STATUS2.equals(status)){
            return true;
        }
        return false;
    }
}
```

上述写法的优点是，将所有状态相关的逻辑判断收敛到`UserStatus`这个文件中。当新增或者删减状态时，可以顺手把相应的逻辑给修改了，也算一个进步。

**但是，真的会顺手改吗**？

事实证明，大部分需要人来主动操作的事情都是不那么靠谱的，**我们需要将主动变成被动**

**优化2**

怎么弄呢？可以这样修改：

```java
public enum UserStatus{
    STATUS1{
        boolean canDoSomething(){
            return true;
        }
    },
    STATUS2{
        boolean canDoSomething(){
            return true;
        }
    },
    STATUS3{
        boolean canDoSomething(){
            return false;
        }
    },
    STATUS4{
        boolean canDoSomething(){
            return false;
        }
    },
    STATUS5{
        boolean canDoSomething(){
            return false;
        }
    };
    
    abstract boolean canDoSomething();
```

上述写法，通过使用抽象方法，强制校验代码编辑的完整性！如果新增状态，不实现相应的方法，会编译不通过。这样，就成功地**将问题暴露在编译阶段，而不是运行阶段**
