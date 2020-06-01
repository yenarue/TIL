# Vue.js 정복 캠프 6기

출석번호 : 1722

* [강의 자료](https://joshua1988.github.io/vue-camp/)

* [강의 GitHub Repository](https://github.com/joshua1988/vue-camp)




## slot

```html
<!-- ListItem.vue -->
<template>
  <div>
    <li>
      <slot></slot>
    </li>
  </div>
</template>
```

해당 컴포넌트를 사용하는 곳에서 `<slot></slot>` 에 내용을 채워넣을 수 있다..

```html
<!-- App.vue -->
<template>
  <div id="app">
    <ul>
      <list-item>item 1</list-item>
      <list-item>item 2</list-item>
      <list-item>
        <h2>item 3</h2>
        <button>say hi</button>
      </list-item>
    </ul>
  </div>
</template>
```

![](../images/vue-slot.png)

### slot 이름 지정하기

```html
<!-- App.vue -->
<template>
  <div id="app">
    <ul>
      <list-item>
        <p slot="title">item 2</p>
        <img
          slot="image"
          src="imageUrl"
        />
      </list-item>
    </ul>
  </div>
</template>
```

```html
<!-- ListItem.vue -->
<template>
  <div>
    <li>
      <slot name="title"></slot>
      <slot name="image"></slot>
    </li>
  </div>
</template>
```

![](../images/vue-slot-name.png)

