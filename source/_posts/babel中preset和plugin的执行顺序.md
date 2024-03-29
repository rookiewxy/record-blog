
# babel的plugins为什么要在preset之前执行

## 1. 插件和预设的区别
### 1.1 插件
##### 1.1.1 插件的定义

插件是 Babel 的一个重要概念，它是用来扩展 Babel 转换功能的工具。插件是一个函数，它接收一个 Babel 解析器解析出来的 AST（抽象语法树）并返回一个新的 AST。在 Babel 的转换过程中，每个插件都会对应一个或多个转换规则，用来对 AST 进行修改或转换。

##### 1.1.2 插件和预设的区别

插件和预设的区别在于它们的作用范围和使用方式。插件是针对具体的转换规则进行编写的，每个插件只能转换一个或少数几个语法特性。而预设是一组插件的集合，它们可以一次性地完成一组转换任务。预设是由一组插件组成的，每个插件都有自己的转换规则，它们被一起打包成一个预设，以便于在 Babel 中进行统一管理和使用。

在使用 Babel 进行转换时，插件和预设的执行顺序也有所不同。Babel 的插件是按照在配置文件中声明的顺序依次执行的，而预设中包含的插件则是按照预设的顺序依次执行的。因此，在配置文件中声明的插件会先于预设中包含的插件执行。这也就解释了为什么在 Babel 的配置文件中，plugins 需要在 presets 之前声明的原因。
### 1.2 预设
##### 1.2.1 预设是什么

预设（preset）是一组预先定义好的插件集合，用于实现特定的功能或者转换特定的语法。它们被打包在一起，可以通过一个简单的名称来引用。预设可以包含多个插件，并且它们按照特定的顺序执行。使用预设可以方便地应用一组常用的插件，而不必手动一个一个地添加它们。

##### 1.2.2 预设的使用

使用预设非常简单，只需要在 Babel 配置文件的 `presets` 字段中指定预设的名称即可。例如，如果要使用 Babel 官方提供的 `env` 预设，可以这样配置：

```javascript
{
  "presets": ["@babel/preset-env"]
}
```

这样就会自动加载 `env` 预设中定义的所有插件，实现了对当前环境下的语法特性的转换。

需要注意的是，预设和插件的执行顺序是固定的，预设会在插件之前执行。这是因为预设是一组插件的集合，而插件则是单独的功能模块。如果插件在预设之前执行，可能会出现一些无法预料的问题，因为插件的功能往往是相互独立的。因此，预设必须在插件之前执行，以确保插件的功能不会被破坏。

## 2. 执行顺序的问题
### 2.1 插件在preset之前执行的原因
##### 2.1.1 插件在preset之前执行的原因

在使用Babel进行代码转换时，我们可以通过配置文件中的`plugins`和`presets`来指定需要使用的插件和预设。插件是Babel的基本功能单元，它们会对代码进行特定的转换操作，可以实现一些特定的功能，例如转换ES6语法、添加polyfill等。而预设则是一组预定义的插件集合，可以方便地进行一次性配置。

在Babel的转换过程中，插件和预设的执行顺序非常重要。事实上，Babel会先执行插件，再执行预设。这是因为插件可以覆盖预设的部分功能，从而实现更加灵活的转换。如果预设先执行，插件就无法覆盖它们的转换操作了。

举个例子，假设我们有一个代码文件需要进行转换，其中包含ES6的箭头函数和类定义。我们可以通过以下配置来进行转换：

```javascript
{
  "plugins": [
    "@babel/plugin-transform-arrow-functions"
  ],
  "presets": [
    "@babel/preset-env"
  ]
}
```

在这个配置中，我们指定了一个插件`@babel/plugin-transform-arrow-functions`和一个预设`@babel/preset-env`。由于插件在预设之前执行，所以箭头函数的转换操作会先被执行，然后再由预设处理其他ES6语法的转换。

总之，插件在preset之前执行的原因是为了确保插件可以覆盖预设的转换操作，从而实现更加灵活的转换。
### 2.2 插件在preset之后执行的影响
##### 3.2 插件在preset之前执行的影响

1. 插件在preset之前执行可以更灵活地对代码进行转换。由于preset是一组预设的插件集合，其转换顺序已经固定，如果插件在preset之后执行，可能会受到preset中插件顺序的限制。而插件在preset之前执行，可以根据需要自由组合插件，实现更加细致的代码转换。

2. 插件在preset之前执行可以更加精细地控制代码转换的过程。由于插件在preset之前执行，可以根据需要自由组合插件，因此可以更加精细地控制代码转换的过程。例如，可以根据代码的具体情况选择不同的插件组合，以达到更好的转换效果。

3. 插件在preset之前执行可以更好地适应不同的需求。由于插件在preset之前执行，可以根据需要自由组合插件，因此可以更好地适应不同的需求。例如，可以根据不同的项目需求选择不同的插件组合，以达到最佳的转换效果。同时，由于插件在preset之前执行，还可以根据不同的代码风格进行定制化的转换，以满足不同的项目需求。
### 2.3 示例
##### 3.1 示例

以下是一个使用 Babel 转换 ES6 代码的示例，其中包含了一个插件和一个预设：

```javascript
// babel.config.js
module.exports = {
  plugins: [
    "transform-decorators-legacy"
  ],
  presets: [
    [
      "@babel/preset-env",
      {
        "targets": {
          "chrome": "58",
          "ie": "11"
        },
        "useBuiltIns": "usage",
        "corejs": "3.6.5"
      }
    ]
  ]
};
```

这个示例中，我们首先使用了一个插件 `transform-decorators-legacy`，它会将类的装饰器语法转换成 ES5 代码。然后我们使用了一个预设 `@babel/preset-env`，它会根据我们指定的目标浏览器和使用的 ES6 特性，自动转换成对应的 ES5 代码。

需要注意的是，插件的执行顺序优先于预设。也就是说，Babel 会先执行插件，再执行预设。在这个示例中，先执行了插件 `transform-decorators-legacy`，再执行了预设 `@babel/preset-env`。

这是因为插件的作用是对代码进行某些特定的转换，而预设则是一组插件的集合，用于对代码进行更全面的转换。如果预设先执行，那么插件可能就会失效。因此，Babel 设计了插件先于预设的执行顺序，以确保插件的转换能够正确地应用到代码中。

## 3. 总结