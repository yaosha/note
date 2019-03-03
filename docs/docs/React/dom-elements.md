## dom-elements

在React中，除了`aria-*`和`data-*`，所有属性包括事件处理器应该使用驼峰命名。

### 属性的使用

* checked
  * 对应HTML属性：checked
  * 支持：type为`checkbox`和`radio`的input
  * 目的：设置是否选中
  * defaultChecked：设置初次加载是否选中
* className
  * 对应HTML属性：class
  * 支持：所有常规DOM和SVG元素
  * 目的：定义CSS类
* dangerouslySetInnerHTML
  * 对应HTML属性：innerHTML
  * 使用有风险,易被跨站点脚本（XSS）攻击，HTML字符串需转义使用
  * 使用：传递对象`{__html: 'First &middot; Second'}` 
  ```js
  function createMarkup() {
    return {
      __html: 'First &middot; Second'};
    }

    function MyComponent() {
      return (<div dangerouslySetInnerHTML={createMarkup()} />);
    }
  ```
* htmlFor
  * 对应HTML属性：for
* onChange
  * 对应HTML属性：onchange
  * 目的：监听输入框变更
* selected
  * 对应HTML属性：selected
  * 支持：`option`标签
  * 目的：设置是否选中
* style
  * 对应HTML属性：style
  * 支持：所有常规DOM和SVG元素
  * 不建议使用style定义组件样式，使用`className`代替，`style`用来动态修改组件样式
  * 使用：接受对象，key必须使用驼峰命名
  ```js
  const divStyle = {
    color: 'blue',
    backgroundImage: 'url(' + imgUrl + ')',
  };

  function HelloWorldComponent() {
    return <div style={divStyle}>Hello World!</div>;
  }
  ```
* suppressContentEditableWarning
* suppressHydrationWarning
* value
  * 对应HTML属性：value
  * 支持：`<input>`和`<textarea>`
  * 目的：设置组件值
  * defaultValue：设置初次加载时组件值

### 所有支持的HTML属性
  
React 16完全支持所有标准或自定义DOM属性。

React支持的DOM属性
```
accept acceptCharset accessKey action allowFullScreen alt async autoComplete
autoFocus autoPlay capture cellPadding cellSpacing challenge charSet checked
cite classID className colSpan cols content contentEditable contextMenu controls
controlsList coords crossOrigin data dateTime default defer dir disabled
download draggable encType form formAction formEncType formMethod formNoValidate
formTarget frameBorder headers height hidden high href hrefLang htmlFor
httpEquiv icon id inputMode integrity is keyParams keyType kind label lang list
loop low manifest marginHeight marginWidth max maxLength media mediaGroup method
min minLength multiple muted name noValidate nonce open optimum pattern
placeholder poster preload profile radioGroup readOnly rel required reversed
role rowSpan rows sandbox scope scoped scrolling seamless selected shape size
sizes span spellCheck src srcDoc srcLang srcSet start step style summary
tabIndex target title type useMap value width wmode wrap
```

所有SVG属性也完全支持
```
accentHeight accumulate additive alignmentBaseline allowReorder alphabetic
amplitude arabicForm ascent attributeName attributeType autoReverse azimuth
baseFrequency baseProfile baselineShift bbox begin bias by calcMode capHeight
clip clipPath clipPathUnits clipRule colorInterpolation
colorInterpolationFilters colorProfile colorRendering contentScriptType
contentStyleType cursor cx cy d decelerate descent diffuseConstant direction
display divisor dominantBaseline dur dx dy edgeMode elevation enableBackground
end exponent externalResourcesRequired fill fillOpacity fillRule filter
filterRes filterUnits floodColor floodOpacity focusable fontFamily fontSize
fontSizeAdjust fontStretch fontStyle fontVariant fontWeight format from fx fy
g1 g2 glyphName glyphOrientationHorizontal glyphOrientationVertical glyphRef
gradientTransform gradientUnits hanging horizAdvX horizOriginX ideographic
imageRendering in in2 intercept k k1 k2 k3 k4 kernelMatrix kernelUnitLength
kerning keyPoints keySplines keyTimes lengthAdjust letterSpacing lightingColor
limitingConeAngle local markerEnd markerHeight markerMid markerStart
markerUnits markerWidth mask maskContentUnits maskUnits mathematical mode
numOctaves offset opacity operator order orient orientation origin overflow
overlinePosition overlineThickness paintOrder panose1 pathLength
patternContentUnits patternTransform patternUnits pointerEvents points
pointsAtX pointsAtY pointsAtZ preserveAlpha preserveAspectRatio primitiveUnits
r radius refX refY renderingIntent repeatCount repeatDur requiredExtensions
requiredFeatures restart result rotate rx ry scale seed shapeRendering slope
spacing specularConstant specularExponent speed spreadMethod startOffset
stdDeviation stemh stemv stitchTiles stopColor stopOpacity
strikethroughPosition strikethroughThickness string stroke strokeDasharray
strokeDashoffset strokeLinecap strokeLinejoin strokeMiterlimit strokeOpacity
strokeWidth surfaceScale systemLanguage tableValues targetX targetY textAnchor
textDecoration textLength textRendering to transform u1 u2 underlinePosition
underlineThickness unicode unicodeBidi unicodeRange unitsPerEm vAlphabetic
vHanging vIdeographic vMathematical values vectorEffect version vertAdvY
vertOriginX vertOriginY viewBox viewTarget visibility widths wordSpacing
writingMode x x1 x2 xChannelSelector xHeight xlinkActuate xlinkArcrole
xlinkHref xlinkRole xlinkShow xlinkTitle xlinkType xmlns xmlnsXlink xmlBase
xmlLang xmlSpace y y1 y2 yChannelSelector z zoomAndPan
```
