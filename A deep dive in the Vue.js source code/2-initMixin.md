  https://medium.com/@oneminutejs/a-deep-dive-in-the-vue-js-source-code-the-initmixin-function-part-1-dc951603a3c
fThis is the second post in a series taking a deep dive in the Vue.js source code. The ._init method is defined in the initMixin function. initMixin is called and passed the Vue object constructor function along with a list of other function calls immediately after the Vue object constructor function is defined:
.init 方法在 initMixin 函数中定义  
initMixin 在定义Vue构造函数 和其他一系列函数函数被立刻调用 并传入Vue构造函数
initMixin(Vue);
stateMixin(Vue);
eventsMixin(Vue);
lifecycleMixin(Vue);
renderMixin(Vue);
The initMixin function is a simple function that takes the Vue object constructor function as a parameter and sets the ._init method on its prototype:
initMixin 函数是一个将Vue 构造函数作为参数的简单的函数 且设置 .init 方法在他的原型上
var uid$3 = 0;
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
    var vm = this;
    // a uid
    vm._uid = uid$3++;
var startTag, endTag;
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = "vue-perf-start:" + (vm._uid);
      endTag = "vue-perf-end:" + (vm._uid);
      mark(startTag);
    }
// a flag to avoid this being observed
    vm._isVue = true;
    // merge options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options);
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      );
    }
    /* istanbul ignore else */
    {
      initProxy(vm);
    }
    // expose real self
    vm._self = vm;
    initLifecycle(vm);
    initEvents(vm);
    initRender(vm);
    callHook(vm, 'beforeCreate');
    initInjections(vm); // resolve injections before data/props
    initState(vm);
    initProvide(vm); // resolve provide after data/props
    callHook(vm, 'created');
/* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false);
      mark(endTag);
      measure(("vue " + (vm._name) + " init"), startTag, endTag);
    }
if (vm.$options.el) {
      vm.$mount(vm.$options.el);
    }
  };
}
A variable uid$3 is set to 0 outside the function in the top level scope to act as a counter that is incremented and set as a property on the Vue instance each time a new Vue instance is created and the ._init method is called.
在顶层作用域设置变量 uid$3 为0 作为计数器  在每次创建 新的vue 实例 且 ._init  方法被调用时自增
var uid$3 = 0
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
    [. . . .]
    // a uid
    vm._uid = uid$3++;
    [. . . .]
  }
}
The ._init method sets a helper variable as this. This approach is identical to setting self = this by saving the current context of the this keyword to a variable for later use.
._init 方法设置了一个this 的辅助变量
这种方式和设置 self= this 一样， 将this关键字当前上下文保存为变量， 以供以后使用。
Vue.prototype._init = function (options) {
  var vm = this;
  [. . . .]
}
Next, the ._init method sets up a performance check.
然后 ._init方法设置了一个性能检测
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
    [. . . .]
    var startTag, endTag;
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = "vue-perf-start:" + (vm._uid);
      endTag = "vue-perf-end:" + (vm._uid);
      mark(startTag);
    }
    [. . . .]
  }
}

The ._init method declares two variables: startTag and endTag.
声明了两个变量 ：startTag 和endTag
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
    [. . . .]
    var startTag, endTag;
    [. . . .]
  }
}
Then you may notice an odd comment:
然后你可能注意到一个奇怪的注释
/* istanbul ignore if */
Istanbul is “Yet another JS code coverage tool that computes statement, line, function and branch coverage with module loader hooks to transparently add coverage when running tests.” The coverage hint tells Istanbul to ignore the if statement.
The if statement first checks whether the current environment is not the production environment.
Istanbul 是： 另一个JS代码覆盖工具 当运行测试时 通过模块loader钩子 透明的添加覆盖率 
这个覆盖提示告诉Istanbul 忽略这个if语句
这个if语句首先检查当前环境是否生产环境
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
    [. . . .]
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = "vue-perf-start:" + (vm._uid);
      endTag = "vue-perf-end:" + (vm._uid);
      mark(startTag);
    }
    [. . . .]
  }
}
The if statement then checks whether config.performance is set to true:
然后检测 config.performance 是否设置为true
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
    [. . . .]
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = "vue-perf-start:" + (vm._uid);
      endTag = "vue-perf-end:" + (vm._uid);
      mark(startTag);
    }
    [. . . .]
  }
}
This leads us to a quick detour because the config object is set elsewhere and defaulted to false. As the comment introducing the config object’s performance property states, config.performance determines whether Vue records performance:
这导致我们快速绕道因为这个config对象是设置在其他地方 且默认为false
按照 config 对象的 performance 属性状态的注释， config.performance 定义Vue是否记录性能
var config = ({
  [. . . .]
/**
   * Whether to record perf
   */
  performance: false,
[. . . .]
})
Turning back to the Vue.prototype._init method, the if statement next checks for a variable named mark.
回到 Vue.prototype._init 方法 ，if语句接下来检查一个名为mark的变量
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
    [. . . .]
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = "vue-perf-start:" + (vm._uid);
      endTag = "vue-perf-end:" + (vm._uid);
      mark(startTag);
    }
    [. . . .]
  }
}
This leads to another quick detour because the mark variable is defined elsewhere in the source code:
这导致另一个快速绕道 因为mark变量是定义在源码的其他地方。
var mark;
var measure;
{
  var perf = inBrowser && window.performance;
  /* istanbul ignore if */
  if (
    perf &&
    perf.mark &&
    perf.measure &&
    perf.clearMarks &&
    perf.clearMeasures
  ) {
    mark = function (tag) { return perf.mark(tag); };
    measure = function (name, startTag, endTag) {
      perf.measure(name, startTag, endTag);
      perf.clearMarks(startTag);
      perf.clearMarks(endTag);
      perf.clearMeasures(name);
    };
  }
}
However, the mark variable will only be defined under certain circumstances. First, we check to see whether the environment is a browser:
然而 ，mark变量将只会定义在某些情况下
首先 我们检查环境是否是一个浏览器环境
// Browser environment sniffing
var inBrowser = typeof window !== 'undefined';
[. . . .]
{
  var perf = inBrowser && window.performance;
  [. . . .]
}
If the environment is a browser, the && chains the expression together so we check whether window.performance exists and set perf to window.performance if it does:
如果是浏览器环境 && 将表达式链接在一起 所以我们检查window.performance 是否存在  若window.performance 存在将其赋值给 perf 变量
{
  var perf = inBrowser && window.performance;
  [. . . .]
}
Understanding what is going on here requires a quick dive in to the Window interface’s Performance property. According to MDN, the “The Window interface’s performance property returns a Performance object, which can be used to gather performance information about the current document. It serves as the point of exposure for the Performance Timeline API, the High Resolution Time API, the Navigation Timing API, the User Timing API, and the Resource Timing API.” The Performance interface is part of the High Resolution Timing API. It “provides access to performance-related information for the current page.”
为了理解接下来将要发生什么需要快速了解下Window 接口的Performance 属性
根据MDN ，Window接口的performance属性返回一个Performance对象，可用来收集当前document的性能信息
它作为性能时间表API，高分辨率时间API，导航时间API，用户时间API和资源时序API的暴露点。“性能接口是高分辨率时序API的一部分。它“提供访问当前页面的与性能相关的信息。”
mark, measure, clearMarks, and clearMeasures are methods on the Performance object.
  ● The mark method creates “a timestamp in the browser’s performance entry buffer with the given name.”
  ● The measure method creates “a named timestamp in the browser’s performance entry buffer between two specified marks (known as the start mark and end mark, respectively).”
  ● The clearMarks method removes “the given mark from the browser’s performance entry buffer.”
  ● The clearMeasures method removes “the given measure from the browser’s performance entry buffer.”
mark measure clearMarks clearMeasures是Performance 对象上的方法
mark 方法在浏览器的性能输入缓冲区中创建一个 timestamp，基于给定的 name
measure 在浏览器的指定 start mark 和 end mark 间的性能输入缓冲区中创建一个指定的 timestamp
clearMars  从浏览器的性能输入缓冲区中移除给定的mark
clearMeasure  从浏览器的性能缓冲区移除给定的measure

As the Vue.js API explains, if the performance option on config is set to true, it enables “component init, compile, render and patch performance tracing in the browser devtool performance/timeline panel” but only works in development mode and in browsers that support the performance.mark API.”
So let’s take a look back at the mark variable initialization:
据Vue.js API 解释，若Performance 选项设为true它 能启用：‘
浏览器devtool性能/时间轴面板中的组件初始化 编译 渲染 修补程序中的性能跟踪’
但是只在支持performance.mark API的浏览器中 的开发环境工作
{
  var perf = inBrowser && window.performance;
  /* istanbul ignore if */
  if (
    perf &&
    perf.mark &&
    perf.measure &&
    perf.clearMarks &&
    perf.clearMeasures
  ) {
    mark = function (tag) { return perf.mark(tag); };
    measure = function (name, startTag, endTag) {
      perf.measure(name, startTag, endTag);
      perf.clearMarks(startTag);
      perf.clearMarks(endTag);
      perf.clearMeasures(name);
    };
  }
}
So if the perf object exists and has mark, measure, clearMarks, and clearMeasures methods, then Vue sets mark and measure functions.
所以若Performance 对象存在其有mark measure clearMark clearMeasures 方法 ，
vue 会设置mark 和measure 函数
The mark function takes a tag as a parameter and returns a timestamp in the browser’s performance entry buffer with the tag as the name.
mark 函数使用tag 作为参数 返回浏览器的性能输入缓冲区中返回的时间戳
mark = function (tag) { return perf.mark(tag); };
Together with measure, this functions allow us to trace performance in the browser devtool performance/timeline panel.
这个函数和measure 允许我们跟中浏览器开发工具 performance/timeline 中的性性能
Now that we have figured out what the mark function does, we are finally able to turn back to the Vue.prototype._init method and understand what is going on. The code below checks for a development environment, ensures that the performance config option is set to true, and ensures that a markfunction exists. If so, Vue sets a start and end tag and calls the mark function passing the startTag as a parameter. This returns a timestamp in the browser’s performance entry buffer with the tag as the name.
现在我们搞清楚了Mark函数做了什么 我们最后回到Vue.prototype._init 方法中 理解接下来发生什么
下边的代码检查是否开发环境 确保性能配置设置为true 确保Mark函数存在
若如此 Vue设置开始和结束的tag 并将startTag作为参数调用mark函数 
这将在标签作为名称的浏览器性能条目缓冲区中返回一个时间戳。
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
    var vm = this;
    // a uid
    vm._uid = uid$3++;
var startTag, endTag;
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = "vue-perf-start:" + (vm._uid);
      endTag = "vue-perf-end:" + (vm._uid);
      mark(startTag);
    }
    [. . . .]
  }
}
The Vue.prototype._init method next sets a property on the Vue instance object called ._isVue and sets it to true as a flag in order to prevent it from being observed:
// a flag to avoid this being observed
vm._isVue = true;
The Vue.prototype._init method then checks to see whether options were passed as a parameter to the ._init method.
 Vue.prototype._init 方法接下来设置一个属性在vue实例对象上 叫做._isVue 
设置为true 作为一个flag来避免他被观察到（看不懂 怎么观察的）
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
  [. . . .]
// merge options
    if (options && options._isComponent) {
    // optimize internal component instantiation
    // since dynamic options merging is pretty slow, and none of the
    // internal component options needs special treatment.
      initInternalComponent(vm, options);
    } else {
      vm.$options = mergeOptions(
      resolveConstructorOptions(vm.constructor),
      options || {},
      vm
    );
    }
    [. . . .]
  }
}
As you will remember, the options parameter is passed to the Vue object constructor function when you create a new Vue instance — so the ._initmethod which is set on Vue.prototype has access to the parameter:
正如你会记得的，当你创建一个新的Vue实例时，options参数被传递给Vue对象构造函数 - 因此在Vue.prototype上设置的._init方法可以访问参数
function Vue (options) {
  [. . . .]
}
After checking whether any options were passed, the ._init method next checks whether options._isComponent is set to true:
检查完options 是否被传递后 ._init 方法检查options._isComponent 是否设置为true
if (options && options._isComponent) {
  // optimize internal component instantiation
  // since dynamic options merging is pretty slow, and none of the
  // internal component options needs special treatment.
  initInternalComponent(vm, options);
}

options._isComponent is only set to true in one instance throughout the Vue.js source code — in the createComponentInstanceForVnode function:
options._isComponent 是在一实例中唯一设置为true的  在整个vue.js 源码中 
在createComponentInstanceForVnode 函数中
function createComponentInstanceForVnode (
  vnode, // we know it's MountedComponentVNode but flow doesn't
  parent, // activeInstance in lifecycle state
  parentElm,
  refElm
) {
  var options = {
    _isComponent: true,
    parent: parent,
    _parentVnode: vnode,
    _parentElm: parentElm || null,
    _refElm: refElm || null
  };
  // check inline-template render functions
  var inlineTemplate = vnode.data.inlineTemplate;
  if (isDef(inlineTemplate)) {
    options.render = inlineTemplate.render;
    options.staticRenderFns = inlineTemplate.staticRenderFns;
  }
  return new vnode.componentOptions.Ctor(options)
}
We will come back to the createComponentInstanceForVnode function later. Turning back to the ._init method, if options is passed and options._isComponent is true, the ._init method calls the initInternalComponent function and passes vm (i.e., this/Vue instance) and options as parameters:
我们以后会回到createComponentInstanceForVnode函数
回到._init 方法  如果options 被传递了 切 options._isComponent 是true
 ._init方法会调用initInternalComponent 函数 传入vm （this vue实例） 和options 作为参数
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
  if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the         
      // internal component options needs special treatment.
      initInternalComponent(vm, options);
    }
  }
}
The initInternalComponent function is declared elsewhere in the code:
initInternalComponent 函数在代码的其他地方声明如下：
function initInternalComponent (vm, options) {
  var opts = vm.$options = Object.create(vm.constructor.options);
  // doing this because it's faster than dynamic enumeration.
  var parentVnode = options._parentVnode;
  opts.parent = options.parent;
  opts._parentVnode = parentVnode;
  opts._parentElm = options._parentElm;
  opts._refElm = options._refElm;
var vnodeComponentOptions = parentVnode.componentOptions;
  opts.propsData = vnodeComponentOptions.propsData;
  opts._parentListeners = vnodeComponentOptions.listeners;
  opts._renderChildren = vnodeComponentOptions.children;
  opts._componentTag = vnodeComponentOptions.tag;
if (options.render) {
    opts.render = options.render;
    opts.staticRenderFns = options.staticRenderFns;
  }
}
The initInternalComponent function sets a variable called opts and a property on the Vue object called $options equal to a new object with the vm.constructor.options set as __proto__ :
initInternalComponent 函数设置了一个变量叫做opts 在Vue 对象上设置一个属性 叫做$options 值是以vm.constructor.options  为原型的一个新对象
function initInternalComponent (vm, options) {
  var opts = vm.$options = Object.create(vm.constructor.options);
  [. . . .]
}
The initInternalComponent function then sets a number of properties on the new opts object by default. As the comment explains, it is faster to do this than dynamically enumerate and add properties.
initInternalComponent 函数然后在新的 opts 对象上 设置了多个默认属性
function initInternalComponent (vm, options) {
  [. . . .]
  // doing this because it's faster than dynamic enumeration.
  var parentVnode = options._parentVnode;
  opts.parent = options.parent;
  opts._parentVnode = parentVnode;
  opts._parentElm = options._parentElm;
  opts._refElm = options._refElm;
var vnodeComponentOptions = parentVnode.componentOptions;
  opts.propsData = vnodeComponentOptions.propsData;
  opts._parentListeners = vnodeComponentOptions.listeners;
  opts._renderChildren = vnodeComponentOptions.children;
  opts._componentTag = vnodeComponentOptions.tag;
  [. . . .]
}
Finally, the initInternalComponent function sets two rendering related properties on the opts object if the options.render property is true:
最后 initInternalComponent 函数设置了两个渲染相关的属性在opts对象上 如果
optios.render 属性是true
function initInternalComponent (vm, options) {
  [. . . .]
  if (options.render) {
    opts.render = options.render;
    opts.staticRenderFns = options.staticRenderFns;
  }
}
Jumping out of the initInternalComponent function and back into the initMixin function, if options or options._isComponent are false, initMixinsets a $options property to the results of calling the mergeOptions function and passing three parameters: (1) a function called resolveConstructorOptions with vm.constructor (the constructor of the Vue instance) as a parameter; (2) options or an empty object; and (3) vm (the Vue instance).
跳出initInternalComponent函数 回到initMixin 函数
如果options 或 options._isComponent 是false  ，initMixin 会设置$options 属性 为调用 mergeOptions 函数且传入三个参数 1 一个函数叫resolveConstructorOptions 以vm.constructor (vue实例的构造函数)作为参数
2 options 或空对象 
3 vm (vm 实例)
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
  [. . . .]
// merge options
    if (options && options._isComponent) {
    // optimize internal component instantiation
    // since dynamic options merging is pretty slow, and none of the
    // internal component options needs special treatment.
      initInternalComponent(vm, options);
    else {
      vm.$options = mergeOptions(
      resolveConstructorOptions(vm.constructor),
      options || {},
      vm
    );
    }
    [. . . .]
  }
}
We will dive in to the mergeOptions function and keep working through initMixin in the next post. 
我们将潜入mergeOptions函数，并继续在下一篇文章中介绍initMixin