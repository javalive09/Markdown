# **多屏按键事件分发逻辑**
Android10完善了多屏按键的支持，3.0UI主框架也有明确的多屏显示及交互需求，所以我们有必要对android的多屏按键事件分发逻辑做一个完整的梳理。
# 一.InputManagerService启动流程
如下：
![](.gitbook/assets/StartInputManagerService-0.png)

# 二.InputReader从EentHub读取事件流程
如下：
![](.gitbook/assets/InputReader-0.png)