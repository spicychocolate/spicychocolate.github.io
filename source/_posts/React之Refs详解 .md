---
title: React之Refs详解
categories: 
 - 原理
tags:
 - React
 - Refs
---
> 本文主要涉及`Refs`与`DOM`, `Refs`转发

## Refs使用场景
- 管理焦点，文本选择或媒体播放
- 触发强制动画
- 集成第三方`DOM`库

### 创建Refs
Refs创建有两种方式：
- 通过回调`Refs`创建
- 使用`React.createRef()`创建(`React 16.3` 版本引入)


#### 回调`Refs`
通过`ref`回调函数获取元素DOM,代码如下：
```typescript
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInputRef = null;


    this.focusTextInput = () => {
      // 使用原生 DOM API 使 text 输入框获得焦点
      if (this.textInputRef) this.textInputRef.focus();
    };
  }

  componentDidMount() {
    // 组件挂载后，让文本框自动获得焦点
    this.focusTextInput();
  }

  render() {
    // 使用 `ref` 的回调函数将 text 输入框 DOM 节点的引用存储到 React
    // 实例上（比如 this.textInputRef
    return (
      <div>
        <input
          type="text"
          ref={ref => this.textInputRef = ref}
        />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```
获取子组件的元素DOM, 可以通过给子组件传获取ref的回调props获得，代码如下
```typescript
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} ></input>
    </div>
  );
}

class Parent extends React.Component {
 constructor(props) {
    super(props);
    this.inputElement = null;
  }

  componentDidMount() {
     if (this.inputElement) console.log(this.inputElement)
  }

  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}
      />
    );
  }
}
```
  
#### `React.createRef()`
`React.createRef()` 创建一个能够通过 `ref` 属性附加到 `React` 元素的 `ref`
```typescript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```
当 `ref` 被传递给 `render` 中的元素时，对该节点的引用可以在 `ref` 的 `current` 属性中被访问
```typescript
const node = this.myRef.current;
```
在极少数情况下，开发中需要在父组件中引用子组件的`DOM`节点，虽然可以向子组件添加`ref`,但是只能获取到子组件实例而不是`DOM`节点。在这种情况下可以使用**`Refs`转发**。**`Refs`转发使组件可以像暴露自己的ref一样暴露子组件的ref**。
先看代码：
```typescript
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.buttonRef = React.createRef();
  }

  componentDidMount() {
     console.log(this.buttonRef)
  }

  render() {
    return <FancyButton ref={this.buttonRef}>Click me!</FancyButton>;
  }
}

```
`React.forwardRef` 接受一个渲染函数，其接收 `props` 和 `ref` 参数并返回一个 `React` 节点
这样，就可以通过`FancyButton`组件获取底层`DOM`节点`button`的`ref`
上述示例主要步骤如下：
1.通过调用 `React.createRef` 创建了一个 `React ref` 并将其赋值给 `ref` 变量。
2.通过指定 `ref` 为 `JSX` 属性，将其向下传递给 `<FancyButton ref={ref}>`。
3.`React` 传递 `ref` 给 `fowardRef` 内函数 `(props, ref) => ...`，作为其第二个参数。
4.向下转发该 `ref` 参数到 `<button ref={ref}>`，将其指定为 JSX 属性。
5.当 `ref` 挂载完成，`ref.current` 将指向 `<button> DOM `节点。

### `Refs`转发在高阶组件(`HOC`)中的应用
以下代码为输出组件 `props` 到控制台的`HOC`
```typescript
function logProps(WrappedComponent) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      return <WrappedComponent {...this.props} />;
    }
  }

  return LogProps;
}
```
"logProps"HOC 透传（`pass through`）所有 `props` 到其包裹的组件, 但是`ref`将不会透传下去, 因为 `ref` 不是 `prop` 属性
如果对 `HOC` 添加 `ref`，该 `ref` 将引用最外层的容器组件，而不是被包裹的组件
```typescript
import FancyButton from './FancyButton';

const ref = React.createRef();

// 导入的 FancyButton 组件是高阶组件（HOC）LogProps。
// 尽管渲染结果将是一样的，
// 但 ref 将指向 LogProps 而不是内部的 FancyButton 组件！
// 这意味着不能调用例如 ref.current.focus() 这样的方法
<FancyButton
  label="Click Me"
  handleClick={handleClick}
  ref={ref}
/>;
```
在这种情况下可以使用 `React.forwardRef API` 明确地将 `ref` 转发到内部的 `FancyButton` 组件。
```typescript
function logProps(Component) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      const {forwardedRef, ...rest} = this.props;

      // 将自定义的 prop 属性 “forwardedRef” 定义为 ref
      return <Component ref={forwardedRef} {...rest} />;
    }
  }

  // 注意 React.forwardRef 回调的第二个参数 “ref”。
  // 可以将其作为常规 prop 属性传递给 LogProps，例如 “forwardedRef”
  // 然后它就可以被挂载到被 LogPros 包裹的子组件上。
  return React.forwardRef((props, ref) => {
    return <LogProps {...props} forwardedRef={ref} />;
  });
}
```
全文总结：
> 元素`Dom`可以使用回调`ref`函数和`React.createRef`获取, 子组件元素`Dom`可以给子组件传`ref`的`props`获取, 还可以使用`React.forwardRef`转发`ref`的形式获取

最后可以通过下面的代码加强理解：
<iframe src="https://codesandbox.io/embed/reactzhirefsxiangjie-rggc3?autoresize=1&fontsize=14&hidenavigation=1&view=editor" title="React之Refs详解" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"></iframe>
