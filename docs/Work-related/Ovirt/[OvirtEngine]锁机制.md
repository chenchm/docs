## EngineLock

OvirtEngine是通过 **LockManager**  接口，提供对 **command** 执行时对实际进行资源的加解锁，确保并发执行时的安全性。根据应用的特点，提供了两种类型的锁（ **独占锁** 和 **共享锁** ），封装在了 **EngineLock** 对象中。

```java
private Map<String, Pair<String, String>> exclusiveLocks;
private Map<String, Pair<String, String>> sharedLocks;

public EngineLock() {

}

public EngineLock(Map<String, Pair<String, String>> exclusiveLocks, Map<String, Pair<String, String>> sharedLocks) {
    this.exclusiveLocks = exclusiveLocks;
    this.sharedLocks = sharedLocks;
}
```

独占锁 和 共享锁 都可以同时锁定多个资源，采用了 **Map<String, Pair<String, String>>** 的数据结构，其中外层 Map 的 Key，存放了 加锁资源的 ID，例如虚拟机 ID。**Pair Map Key** 存放了锁定资源组的类型，如VM，**Pair Map Value** 中存放了锁竞争失败时给出的错误提示信息。

资源组的类型如下：

| 序号 | 可锁定资源组类型名称 | 描述               |
| ---- | -------------------- | ------------------ |
| 1    | POOL                 | 池                 |
| 2    | VDS                  | 主机               |
| 3    | VDS_INIT             | 主机初始化         |
| 4    | VDS_FENCE            | 电源管理           |
| 5    | VM                   | 虚拟机             |
| 6    | TEMPLATE             | 模版               |
| 7    | DISK                 | 磁盘               |
| 8    | VM_DISK_BOOT         | 引导盘             |
| 9    | VM_NAME              | 虚拟机名称         |
| 10   | NETWORK              | 网络               |
| 11   | STORAGE              | 存储               |
| 12   | STORAGE_CONNECTION   | 存储连接           |
| 13   | REGISTER_VDS         | 注册主机           |
| 14   | VM_SNAPSHOTS         | 虚拟机快照         |
| 15   | GLUSTER              | 卷                 |
| 16   | USER_VM_POOL         | 池分配虚拟机给用户 |
| 17   | REMOTE_TEMPLATE      | 导出模版           |
| 18   | REMOTE_VM            | 导出虚拟机         |

在AbstractCommand中对获得独占锁方法getExclusiveLocks()和获得共享方法 getSharedLocks()都给出了默认的空实现，子类可以重写这两个方法来添加锁定的资源。

## 锁资源竞争过程

在Ovirt的command类执行真的逻辑之前会对command的执行的条件进行初始化和校验工作，其中就涉及了对线程共享资源的限制，在执行命令前会尝试去获得锁，该方法在AbstractCommand中的实现如下：

```java
public boolean acquireLock() {
    //从parameters中获得锁属性，如果parameters没有相应的参数，则使用默认值
    //默认值包括锁范围为none，当锁竞争失败时不等待获取锁
    //可以通过重写applyLockProperties方法对锁的属性进行修改
    LockProperties lockProperties = getLockProperties();
    boolean result = true;
    
    //如果锁范围为none则不去尝试获得锁，视为获得锁成功
    if (!Scope.None.equals(lockProperties.getScope())) {
        //如果锁的范围为Execution，将releaseLocksAtEndOfExecute设为true
        //releaseLocksAtEndOfExecute表示在command的Execute阶段结束释放锁
        //command的多个阶段在这里不进行赘述
        releaseLocksAtEndOfExecute = Scope.Execution.equals(lockProperties.getScope());
        
        //判断是否等待获得锁，默认值为false
        if (!lockProperties.isWait()) {
            //由于默认值的原因，ovirt中大部分逻辑发生在这里
            //当发生锁竞争时，放弃获得锁，command执行失败
            result = acquireLockInternal();
        } else {
            //当发生锁竞争时，等待获得锁，然后继续执行commnad
            acquireLockAndWait();
        }
    }
    return result;
}
```

由于acquireLockAndWait()方法与acquireLockInternal()方法逻辑上的最主要的区别只有锁竞争失败时是否继续等待获得锁（共享锁不需要等待），所以这里就只对主要执行逻辑acquireLockInternal()方法进行展开。

```java
protected boolean acquireLockInternal() {
    //如果当前的command没有锁，则创建一个新锁，否则很可能已经获得了父命令的锁。
    if (context.getLock() == null) {
        //没有锁则构建EngineLock
        EngineLock lock = buildLock();
        //如果构建的锁不为空才继续尝试获得锁
        //即如果子类没有重写getExclusiveLocks()或者getSharedLocks()来添加锁，则视为获得锁成功
        if (lock != null) {
            //通过LockManager来尝试获得锁
            Pair<Boolean, Set<String>> lockAcquireResult = getLockManager().acquireLock(lock);
            if (lockAcquireResult.getFirst()) {
                log.info("Lock Acquired to object '{}'", lock);
                //获得锁成功后将context锁设置为构造的锁
                context.withLock(lock);
            } else {
                log.info("Failed to Acquire Lock to object '{}'", lock);
                getReturnValue().getValidationMessages()
                    .addAll(extractVariableDeclarations(lockAcquireResult.getSecond()));
                return false;
            }
        }
    }
    return true;
}

private EngineLock buildLock() {
    EngineLock lock = null;
    //这个两个方法默认返回值为null，由子类重写来加锁
    Map<String, Pair<String, String>> exclusiveLocks = getExclusiveLocks();
    Map<String, Pair<String, String>> sharedLocks = getSharedLocks();
    //如果子类没有重写其中任何一个方法，则不进行锁的构建
    if (exclusiveLocks != null || sharedLocks != null) {
        lock = new EngineLock(exclusiveLocks, sharedLocks);
    }
    return lock;
}
```

从上面的代码可以看，出如果当前的command拥有锁，则说明使用父command的锁；如果当前command没有锁，则构建新的锁，锁的构建与子类是否重写了getExclusiveLocks()或者getSharedLocks()来添加锁有关，若没重写任何方法来添加锁则视为获得锁成功，否则调用LockManager的**acquireLock**方法来尝试获得锁。

```java
public Pair<Boolean, Set<String>> acquireLock(EngineLock lock) {
    log.debug("Before acquiring lock '{}'", lock);
    //这里的globalLock为JUC的ReentrantLock对象，是一个全局锁
    //之所以称为全局锁，是因为所有运行的command调用此方法时都会等待获得globalLock，然后再执行后续操作
    //锁的粒度比较大，因此称为全局锁
    globalLock.lock();
    try {
        //获得了锁的command才能够执行acquireLockInternal方法
        return acquireLockInternal(lock);
    } finally {
        globalLock.unlock();
    }
}
```

这里使用了JUC的ReentrantLock来实现同步，保证多个commnad线程执行此方法时，只有一个线程在执行acquireLockInternal()方法。

```java
private Pair<Boolean, Set<String>> acquireLockInternal(EngineLock lock) {
    boolean checkOnly = true;
    //这里执行了两次循环，第一次checkOnly为true,第二次checkOnly为false
    //即第一次只校验是否能够插入锁，第二次才去插入锁
    //这样做的目的是为了避免如果校验和插入同时进行，如果后续校验失败需要对之前插入的锁进行删除
    for (int i = 0; i < 2; i++) {
        if (lock.getSharedLocks() != null) {
            for (Entry<String, Pair<String, String>> entry : lock.getSharedLocks().entrySet()) {
                //将共享锁放入hashMap缓存,并返回是否获成功获得锁
                Pair<Boolean, Set<String>> result =
                    insertSharedLock(buildHashMapKey(entry), 
                                     entry.getValue().getSecond(), checkOnly);
                if (!result.getFirst()) {
                    log.debug("Failed to acquire lock. Shared lock is taken for key '{}'," + 
                              "value '{}'",
                              entry.getKey(),
                              entry.getValue().getFirst());
                    return result;
                }
            }
        }
        if (lock.getExclusiveLocks() != null) {
            for (Entry<String, Pair<String, String>> entry : lock.getExclusiveLocks().entrySet()) {
                Pair<Boolean, Set<String>> result =
                    //将独占锁放入hashMap缓存,并返回是否获成功获得锁
                    insertExclusiveLock(buildHashMapKey(entry), 
                                        entry.getValue().getSecond(), checkOnly);
                if (!result.getFirst()) {
                    log.debug("Failed to acquire lock. Exclusive lock is taken for key '{}'," +
                              "value '{}'",
                              entry.getKey(),
                              entry.getValue().getFirst());
                    return result;
                }
            }
        }
        checkOnly = false;
    }
    log.debug("Success acquiring lock '{}' succeeded ", lock);
    return LOCK_INSERT_SUCCESS_RESULT;
}
```

上面的代码使用了两次循环来执行锁插入操作（第一次只检查能否插入，第二才真正进行插入），目的是当有多个锁需要检查的话，避免了后续检查失败时要对之前插入的锁进行删除。而insertSharedLock()和insertExclusiveLock()真正获得锁的方法，而锁获得的逻辑十分简单，使用MapEntry的key以及Pair的第一个值构成hashMap的的key，Pair的第二值作为value存入HashMap缓存。区别两者对于获得锁成功的判断不同：

- 独占锁：如果对应的key在HashMap中不存在，将key和value存入HashMap，视为获得锁成功。如果对应的key在HashMap中存在，视为获得锁失败。
- 共享锁：如果对应的key在HashMap中不存在，将key和value存入HashMap，视为获得锁成功。如果对应的key在HashMap中存在，判断锁对象是不是独占锁，如果是独占锁，视为获得锁失败，否则将共享锁的count数加一。

到此，锁竞争的逻辑就结束了，当获得锁失败后，命令的校验也就失败了，命令也就执行失败了。

## 锁的释放

锁的释放在AbstractCommand的*freeLock()*方法中实现，如下：

```java
public void freeLock() {
    // 如果context锁不为空，进行释放锁的工作
    if (context.getLock() != null) {
        // 调用LockManager的releaseLock方法释放当前锁
        getLockManager().releaseLock(context.getLock());
        log.info("Lock freed to object '{}'", context.getLock());
        context.withLock(null);
        // 释放其他锁自定义的锁
        freeCustomLocks();
    }
}
```

freeLock方法会在以下几种情况下被调用：

- command的校验工作失败，最终会释放校验过程中加的锁
- 锁的范围为Execute，在command的Execute阶段结束后释放锁
- command的所有action结束，在endAction方法中释放锁
- 在command中主动调用freeLock方法

LockManager的releaseLock方法如下：

```java
public void releaseLock(EngineLock lock) {
    log.debug("Before releasing a lock '{}'", lock);
    //使用全局锁
    globalLock.lock();
    try {
        if (lock.getSharedLocks() != null) {
            for (Entry<String, Pair<String, String>> entry : lock.getSharedLocks().entrySet()) {
                //释放共享锁
                releaseSharedLock(buildHashMapKey(entry), entry.getValue().getSecond());
            }
        }
        if (lock.getExclusiveLocks() != null) {
            for (Entry<String, Pair<String, String>> entry : lock.getExclusiveLocks().entrySet()) {
                //释放独占锁
                releaseExclusiveLock(buildHashMapKey(entry));
            }
        }
        //isAwait为true的锁，会在竞争锁失败时挂起
        //这里唤醒等待独占锁的线程开始竞争获得锁
        releasedLock.signalAll();
    } finally {
        globalLock.unlock();
    }
}
```

释放锁的过程也很简单，共享锁释放时将count减1，当count为0时从缓存中删除锁；而独占锁释放时直接将锁从缓存中删除即可。

## 只改变数据库状态的操作锁

OvirtEngine中还存在另一种锁，只是改变Entity在数据库的状态，从而实现限制前台页面操作的功能。例如，前台锁定VM的机制，实际上是将数据库的status字段设置为`ImageLocked`状态，而前台会根据这个状态来限制页面的操作功能。VmHandler中lockVm方法如下：

```java
public static void lockVm(final VmDynamic vm, final CompensationContext compensationContext) {
    TransactionSupport.executeInNewTransaction(() -> {
        // 在修改状态前，先存储vm的快照信息，用于操作失败时恢复数据
        compensationContext.snapshotEntityStatus(vm);
        // 调用lockVm方法修改vm状态，主要就是操作数据库修改status字段
        lockVm(vm.getId());
        compensationContext.stateChanged();
    });
}

public static void lockVm(Guid vmId) {
    Backend.getInstance()
        .getResourceManager()
        .runVdsCommand(VDSCommandType.SetVmStatus,
                       new SetVmStatusVDSCommandParameters(vmId, VMStatus.ImageLocked));
}
```

是否需要加锁，由代码的逻辑决定；而解锁工作一般在command执行成功或command失败时执行。

前台如何实现锁定操作？

在VmListModel的updateActionsAvailability()方法维护着前台按钮是否可以点击的逻辑，例如删除虚拟机按钮的可执行逻辑为：

```java
getRemoveCommand()
    .setExecutable(vmsSelected && VdcActionUtils.canExecutePartially(items,
    VmWithStatusForExclusiveLock.class, VdcActionType.RemoveVm));
```

可以看出是否能点击的主要逻辑在于VdcActionUtils.canExecutePartially()方法

```java
public static boolean canExecutePartially(List<? extends BusinessEntityWithStatus<?, ?>> entities, Class type, VdcActionType action) {
    if (blacklist.containsKey(type)) {
        for (BusinessEntityWithStatus<?, ?> a : entities) {
            if (a.getClass() == type && (!blacklist.get(type).containsKey(a.getStatus()) 
                                    || !blacklist.get(type).get(a.getStatus()).contains(action))) {
                return true;
            }
        }
    } else {
        return true;
    }
    return false;
}
```

可以看出前台维护着一张status与Action之间的黑名单列表，即某些状态下一些Action不能执行，`VMStatus.ImageLocked`的action黑名单如下：

```java
vmBlacklist.put(
                VMStatus.ImageLocked,
                EnumSet.of(
                        VdcActionType.RunVm,
                        VdcActionType.CloneVm,
                        VdcActionType.RunVmOnce,
                        VdcActionType.StopVm,
                        VdcActionType.ShutdownVm,
                        VdcActionType.HibernateVm,
                        VdcActionType.MigrateVm,
                        VdcActionType.RemoveVm,
                        VdcActionType.AddVmTemplate,
                        VdcActionType.ExportVm,
                        VdcActionType.ImportVm,
                        VdcActionType.ChangeDisk,
                        VdcActionType.CreateAllSnapshotsFromVm,
                        VdcActionType.AddVmInterface,
                        VdcActionType.UpdateVmInterface,
                        VdcActionType.RemoveVmInterface,
                        VdcActionType.CancelMigrateVm,
                        VdcActionType.ExtendImageSize,
                        VdcActionType.RebootVm));
```

