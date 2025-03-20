### **一、什么是 Tree Shaking？**

**Tree Shaking** 是一种通过静态代码分析，在打包时移除未使用代码的优化技术。它依赖于 ES Module 的静态结构（`import` 和 `export`），能够在构建时识别并删除未被引用的代码，从而减少最终打包文件的体积。

---

### **二、为什么 Vue2 不能实现 Tree Shaking？**

#### **1. Vue2 的模块化设计问题**

- **整体打包**：Vue2 的核心代码是一个整体打包的库（如 `vue.runtime.esm.js`），所有功能都集中在一个文件中。
    
- **非模块化**：Vue2 的许多功能（如指令、过滤器、全局 API）是通过全局变量或原型链挂载的，无法通过静态分析识别未使用的部分。
    
- **Options API**：Vue2 的 Options API（如 `data`、`methods`、`computed`）是动态绑定的，无法在编译时确定哪些代码被使用。
    

#### **2. 技术限制**

- **CommonJS 为主**：Vue2 主要使用 CommonJS 模块化规范，而 Tree Shaking 依赖于 ES Module 的静态特性。
    
- **副作用代码**：Vue2 的代码中存在一些副作用（如全局注册组件、指令），这些代码无法被 Tree Shaking 移除。
    
#### **示例**

```javascript
// Vue2 的全局 API
Vue.directive('focus', { /* ... */ });
Vue.filter('capitalize', function (value) { /* ... */ });
```
- 这些全局 API 无法被 Tree Shaking 移除，即使项目中未使用它们。
    
---

### **三、Vue3 如何实现 Tree Shaking？**

#### **1. 模块化设计**

- **按需导出**：Vue3 将核心功能拆分为多个独立的模块（如 `reactivity`、`runtime-core`、`compiler`），并通过 ES Module 导出。
    
- **功能解耦**：每个功能模块都可以单独引入，未使用的模块不会被打包。
    
#### **2. 基于 ES Module**

- Vue3 完全基于 ES Module 开发，利用其静态特性实现 Tree Shaking。
    
- 打包工具（如 Webpack、Rollup）可以静态分析 `import` 和 `export`，移除未使用的代码。
    

#### **3. 减少副作用**

- Vue3 的代码设计尽量避免副作用，确保未使用的代码可以被安全移除。
    
- 例如，Vue3 的全局 API（如 `createApp`）是通过显式导入的，而不是挂载到全局变量。
    
#### **示例**

```javascript
// Vue3 的按需导入
import { ref, computed } from 'vue';

export default {
  setup() {
    const count = ref(0);
    const double = computed(() => count.value * 2);
    return { count, double };
  },
};
```

- 如果未使用 `computed`，打包工具会将其移除。
    

---

### **四、Vue3 实现 Tree Shaking 的具体措施**

#### **1. 模块拆分**

Vue3 将核心功能拆分为多个模块：

- `reactivity`：响应式系统。
    
- `runtime-core`：运行时核心。
    
- `compiler`：模板编译器。
    
- `shared`：共享工具函数。
    

#### **2. 按需导入**

开发者可以只导入需要的功能：

```javascript
import { ref, onMounted } from 'vue';
```

- 未导入的功能（如 `watch`、`computed`）不会被打包。

#### **3. 减少全局 API**

Vue3 移除了许多全局 API（如 `Vue.directive`、`Vue.filter`），改为显式导入：

```javascript
import { createApp } from 'vue';
const app = createApp();
app.directive('focus', { /* ... */ });

```
#### **4. 编译优化**

Vue3 的模板编译器会生成更高效的代码，减少不必要的运行时开销。

---

### **五、Tree Shaking 的效果**

#### **1. 打包体积更小**

- Vue3 的基础运行时体积比 Vue2 更小（约 10KB gzip）。
    
- 未使用的功能（如 `v-model`、`transition`）不会被打包。
    
#### **2. 性能更好**

- 更小的代码体积意味着更快的加载速度和解析速度。
    
- 按需加载功能减少了内存占用。
    

#### **示例对比**

- **Vue2**：即使只使用 `ref` 和 `computed`，也会打包整个 Vue2 运行时（约 30KB gzip）。
    
- **Vue3**：只打包使用的功能（如 `ref` 和 `computed`），体积更小（约 5KB gzip）。
    

---

### **六、总结**

|**特性**|**Vue2**|**Vue3**|
|---|---|---|
|**模块化设计**|整体打包，功能耦合|模块化设计，功能解耦|
|**模块规范**|主要使用 CommonJS|完全基于 ES Module|
|**Tree Shaking**|不支持|支持|
|**打包体积**|较大（包含未使用功能）|较小（按需打包）|
|**性能**|较低（加载和解析较慢）|较高（加载和解析更快）|

**Vue3 通过模块化设计、ES Module 支持和减少副作用代码，实现了 Tree Shaking**，从而显著减少了打包体积，提升了性能。这种设计思路对开发其他库和框架具有重要的借鉴意义，尤其是在现代前端工程化中，Tree Shaking 已成为优化打包体积的标配技术。