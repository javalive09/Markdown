# Framework - AMS

## android9.0 Context
使用了装饰器模式
![](.gitbook/assets/Context-0.png)

## android9.0 启动activity
### 1. startActivity
![](.gitbook/assets/StartActivity_Start-0.png)
### 2. ams对wms的控制
![](.gitbook/assets/StartActivity_AMS2WMS-0.png)
### 3. pause back stack
![](.gitbook/assets/StartActivity_pause_5_3_-0.png)
### 4.1 如果进程不存在则创建进程 fork process
![](.gitbook/assets/StartActivity_forktheprocess-0.png)
### 4.2 如果进程存在则使用这个进程 
![](.gitbook/assets/StartActivity_starttheprocess-0.png)
### 5. attach
![](.gitbook/assets/StartActivity_attach-0.png)