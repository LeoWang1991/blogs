#### Question

1. 什么是virtual dom，优劣，为甚virtualdom快
2. virtual dom实现原理
3. React生命周期（最新的 哪些废弃的），react请求应该放在哪个阶段
4. setState 同步还是异步，发生了什么。即react render的过程（diff和conciliation v16是fiber算法）
5. react组件之间的通信
6. react如何实现组件/逻辑上的复用
7. mixin hoc props-render hooks 优劣区别
8. 怎么理解fiber
9. time slice的理解(这个真没听过=。=)
10. redux 工作流 原理（过一遍源码）
11. react-redux如何工作的
12. mobx了解 mobx如何使用 
13. redux mobx差别
14. redux如何进行异步操作（中间件 redux-saga等）
15. redux异步中间件有哪些 优劣
16. react性能优化

参考[2019需要了解的一些React知识点详解](https://juejin.im/post/5d5f44dae51d4561df7805b4)



#### 1 VirtualDOM

VDOM一种编程概念，可以理解为真实DOM对应的虚拟DOM（UI），VDOM包括了真实DOM的所有状态信息，开发者不用直接去操作DOM比如新增一条信息，以往我们需要自己去append(li-dom)，现在我们只要关注和更新VDOM就行了，VDOM变了，真实的DOM也会发生改变，这个过程我们不需要去关注React(ReactDOM)会去做这个操作，称之为协调(reconcilition)。为什么快，vdom是保存在内存中的js对象。

**React操作比操作真是DOM要快？**
参考网上都说操作真实 DOM 慢，但测试结果却比 React 更快，为什么？ - 罗志宇的回答 - 知乎 https://www.zhihu.com/question/31809713/answer/54833728

React通过协调过程把VD的更新反映到DOM中，也会重新渲染DOM，但是React快是因为很多情况直接去操作DOM会引起整个页面的渲染，而React的话会去计算部分更新DOM(变化的VD对应的) 这个时间= diff + render页面的时间，比直接操作DOM多了个diff算法计算的时间，如果说计算出来需要更新的DOM也是整个页面，这种情况React还要慢因为渲染的时间和直接操作的差不多。打个比方：页面中插入一条数据(或者其他操作)，直接操作DOM更改可能引起页面所有元素的重新渲染(比如100个) 时间T1=renderElement(100)，而React通过diff算法(耗时diffTime)计算出前后两次VD的差异即需要更新的内容部分(比如20个)时间T2=diffTime + renderElement(20)。
diffTime因为是纯js计算很快，所以总得来说React渲染要快很多。还有一些比如React实现这个过程会去重复利用现在的节点



**实现原理**

DOM = f(VDOM)。
个人理解：比如`<div>hello</div>` 元素通过一个对象Object来表示,上面有很这个元素的所有信息`{type: 'div', value: 'hello', props: {}, children: []}` 类似这样，然后通过一个函数比如`ReactDOM.render` 将这个渲染为真实的DOM节点。



#### render和setState

setState会触发render函数，返回当前状态时的一个VDOM-tree虚拟DOM树，然后这棵树去上次的状态去比较通过diff算法计算出前后两次VD的差异，然后React将这个差异更新到对应的真实DOM。



#### Diff算法

diff算法的作用在于计算react state或者props更新后新的VDOM(NewTree)和更新前的VDOM(OldTree)的最小差异部分，然后针对该部分对实际的DOM进行渲染而非整个页面进行渲染。所以React之所以高效得益于diff算法性能的优秀。
这个过程称之为[Reconciliation 调和](https://reactjs.org/docs/reconciliation.html) [翻译](https://testudy.cc/tech/2017/06/21/react-25-reconciliation.html)

传统的计算方法复杂度为O(n<sup>3</sup>) n为元素数量，所以传统的计算方式性能很低，而React采用一个启发式O(n)算法基于以下两点：

1. 不同类型元素才生不同不同的树
2. 开发者可以通过`key`值来唯一标识一个子元素

**具体的diff算法**

O(n)的算法简单描述就是，DOM树从上到下从左到右去遍历就行了，遇到不同类型的节点或者类型相同属性不同的元素通过该diff算法去处理就行。

首先react元素可以分为两类吧，一类是DOM元素(div p span等html标签)，一类是自定义的Component组件元素。

首先我们需要比较量NewTree和OldTree的根元素再做如下情况讨论：

**情况1：根节点元素类型不同** tree diff
这就没啥好说的，直接替换好了，销毁原有的，生成新的。比如div变成了p Count组件变成NewCount等。这个判断是通过元素的`type`值来进行的

**情况2：类型相同且是DOM元素** element diff
元素相同，只是props属性(className style data-*自定义prop)更改了，那么React会自动修改这些属性。改完之后，递归处理其子元素。

**情况3：类型相同且是Component组件** component diff
组件更新后还是保持同一个组件实例，当前组件的更新会更新其子元素。
递归子元素，默认情况下，会对DOM节点的子元素进行递归操作，React只需要遍历所有子元素，出现差异时进行修改。
**key的作用**

```html
// 原来
<ul>
  <li>AAA</li>
  <li>BBB</li>
</ul>

// 末尾插入CCC，按照上述遍历头两项逐个比较因为相同所以保持不变，最后一项插入CCC即可
<ul>
  <li>AAA</li>
  <li>BBB</li>
  <li>CCC</li>
</ul>

// 如果在开头插入CCC，逐个比较会发现 这三项全部不一样，则结果处理每个子元素（虽然AAA BBB是相同的）
<ul>
  <li>CCC</li>
  <li>AAA</li>
  <li>BBB</li>  
</ul>
```

所以需要对每个子元素加一个`key`值（key只是在兄弟节点间保持唯一）

所以遍历生成组件列表时候需要给每个组件加`key` 

**权衡一下 其实并不是所有时候都会用到diff算法**

1. 上面的根节点类型不同，就直接替换了 和diff算法没啥关系
2. 相同子元素的key是可预期独一无二的，不标准的Key（比如Math.random()产生的）将导致很多不必要的组件实例和DOM节点被创建，这会导致性能下降和子组件的状态丢失。

