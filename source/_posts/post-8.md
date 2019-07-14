
---
title: emoji在JS中的使用
date: 2019-04-01 16:58:26
tags:
---

### emoji是什么

计算机中的字符都是借助二进制编码来表示，有ASCII，unicode等，ASCII码主要用于显示现代英语和其他西欧语言，其规则是由8个bit组成，第一个bit为0，利用剩余的7个bit组合出128种不同的字符，随着计算机的不断推广，利用单字节编码系统显得力不从心，无法表示出更多的字符，因此需要一个统一的标准，而这就是unicode诞生的原因。

unicode为世界上的所有字符进行了编号，范围从 0x000000 到 0x10FFFF，大多数字符在范围0x0000 到 0xFFFF 之间（即小于 65536），每个字符都有一个 Unicode 编号并且一般用十六进制表示，前置 U+。

emoji也是一个unicode字符，每一个emoji都分配了一个对应的码点（code point），不同的系统对同一个emoji的展示形式也是不同的，如果系统没有实现当前的emoji，那么渲染出来的就是一个系统默认设置的内容
<!-- more -->
### emoji怎么用

在js中我们可以使用\uXXXX表示一个unicode字符，其中XXXX表示的是Unicode码点，只能表示常见的字符(即位于0x0000到0xFFFF范围内的字符)，超出范围的需要用两个双字节表示。

```javascript
//在html中的展示则为 &#x1f61c;
//在es6中支持了以下用法输出emoji和对emoji进行解码

"\u{1f61b}" //😛

String.fromCodePoint("0x1f601") //😛

'😛'.codePointAt(0).toString(16) // 1f61b

```

utf8是mysql中的一种字符集，只支持最长三个字节的 UTF-8字符；utf8mb4可以支持四个字节UTF-8编码 ，但是要使用 MySQL 的这个特性，首先需要把 MySQL 升级到 5.5.3 以上的版本。

emoji一般占四个字节，如果使用MySQL存储Emoji, 需要将数据表的字符集设置为`utf8mb4`, 即`CHARSET=utf8mb4`，这样就能直接将用户输入的emoji信息完整保存在数据库中。

**那么，可以根据一个emojiMapping表，然后利用String.fromCodePoint进行转换然后渲染到视图上，再根据用户的选择再输入框进行拼接，即可实现emoji表情的需求**



### 开发踩坑记录

- 在小程序中，input输入框如果贴底，会出现软键盘直接无视input框的父元素直接贴住input框，需要给input框加一个margin-bottom

![1548069842886](C:\Users\charliemei\AppData\Roaming\Typora\typora-user-images\1548069842886.png)



- 在h5中需要用到**长按**事件去进行评论的删除，这里用到了**touchstart**，**touchmove**，**touchend**进行模拟，首先在**touchstart**中设置一个**定时器**，在定时器中触发注册的**回调函数**，然后在**touchend**中将这个**定时器清除**，如果用户是**长按**，那么就会触发**定时器**，如果是**点击**一下，那么就会在手指离开屏幕的时候触发**touchend**将**定时器清除**而不会被触发。在**屏幕滚动**的过程中由于也会**长按屏幕**，因此需要做一个区分，**滚动**过程会触发**touchmove**事件，因此需要在**touchmove**中**清除定时器**。除此之外，**长按**事件还会触发点击事件，那么需要设置一个**计时器**，记录**touchstart**和**touchend**的间隔时间，如果间隔时间长说明用户希望触发**长按**事件，那么这里就可以通过**preventDefault**将点击事件阻止掉

  ```javascript
  // 移动端长按指令
  Vue.directive("longpress", {
      inserted: function (el) {},
      bind(el, binding) {
          let timeout = undefined;
          const INTERVAL = 200
          let start = 0;
          binding.$$startEvent = function (event) {
              //模拟长按
              timeout = setTimeout(binding.value.bind(null, el, event), INTERVAL);
              start = Date.now()//计时器
              event.stopPropagation();
          };
          binding.$$moveEvent = function (event) {
              clearTimeout(timeout);//避免和滚动事件冲突
              event.stopPropagation();
          };
          binding.$$endEvent = function (event) {
              clearTimeout(timeout);//避免和点击事件冲突
              event.stopPropagation();
              if (Date.now() - start > INTERVAL) {//避免触发祖先元素的点击事件
                  event.preventDefault();
              }
          };
          el.addEventListener("touchstart", binding.$$startEvent, false);
          el.addEventListener("touchmove", binding.$$moveEvent, false);
          el.addEventListener("touchend", binding.$$endEvent, false);
      },
      unbind(el, binding) {
          el.removeEventListener("touchstart", binding.$$startEvent);
          binding.$$startEvent = null;
          el.removeEventListener("touchmove", binding.$$moveEvent);
          binding.$$moveEvent = null;
          el.removeEventListener("touchstart", binding.$$endEvent);
          binding.$$endEvent = null;
      },
      update() {}
  });
  ```




- 在获取滚动高度的时候，用document.documentElement.scrollTop无效，可以通过document.scrollingElement.scrollTop或者window.scrollY获取
- input输入框贴底，会出现软键盘覆盖input框的情况，需要在input框的focus事件中调用scrollIntoView(false)才能正常贴底
- input输入框在ios中调起软键盘的时候页面会被上推，但是blur的时候不会回到原本位置，所以需要判断user-agent是否iphone，在输入框的blur事件中将document.scrollingElement.scrollTop减去对应的高度，否则当用户在页面底部调起输入框时，输入框会因位于页面的区域而出现异常