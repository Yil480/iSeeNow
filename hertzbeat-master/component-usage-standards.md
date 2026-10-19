# HertzBeat 前端组件使用规范

> 适用范围：`web-app` 前端工程（Angular 17 + TypeScript + ng-zorro-antd + @delon + LESS）
> 文档性质：本规范以项目现有配置与代码实践为依据整理。标注 **【强制】** 的条目由 ESLint / Prettier / Stylelint 在构建或提交时自动校验；标注 **【建议】** 的条目为项目现有代码中已被验证的推荐做法。
> 维护原则：只沉淀项目中已经采用或已配置的规则，不引入脱离技术栈的外部规范。

---

## 1. 总体原则

1. **组件优先、页面组合**：页面由组件组合而成；可复用的 UI 与交互必须沉淀为组件，禁止在多个页面复制相同模板。
2. **单一职责**：一个组件只承担一类职责，区分展示组件、交互组件、业务逻辑组件与布局组件。
3. **单向数据流**：父组件通过输入（`@Input`）向下传值，子组件通过输出（`@Output`）上报事件，不反向修改父级状态。
4. **样式默认隔离**：组件样式默认使用 Angular 的 Emulated（模拟隔离）作用域，穿透第三方组件样式时必须以 `:host` 限定。
5. **工具链优先**：命名、格式、引入顺序等以 ESLint / Prettier / Stylelint 配置为最终裁决，人工 Code Review 只关注工具无法覆盖的设计问题。

---

## 2. 命名规范

### 2.1 文件命名

| 对象 | 规则 | 示例 |
| -- | -- | -- |
| 组件文件 | kebab-case，统一以 `.component.ts/.html/.less` 结尾 | `monitor-data-chart.component.ts` |
| 组件目录 | 与组件主体同名，kebab-case | `shared/components/form-field/` |
| 模块文件 | kebab-case，以 `.module.ts` 结尾 | `shared.module.ts` |
| 路由模块 | kebab-case，以 `-routing.module.ts` 结尾 | `alert-routing.module.ts` |
| 单元测试 | 与被测文件同名，以 `.spec.ts` 结尾 | `form-field.component.spec.ts` |
| 服务 | kebab-case，以 `.service.ts` 结尾 | `monitor.service.ts` |
| 管道 | kebab-case，以 `.pipe.ts` 结尾 | `elapsed-time.pipe.ts` |

依据：全工程 `web-app/src/app` 目录结构均遵循上述约定。

### 2.2 组件类命名 **【强制】**

- 类名使用 **PascalCase**。
- 类名必须以下列后缀之一结尾：`Component`、`Directive`、`Base`、`Widget`。

```ts
// 正确
export class ToolbarComponent {}
export class MonitorDataChartComponent {}
export class TestWidget extends ControlWidget {}

// 错误：缺少受允许的后缀
export class Toolbar {}
```

依据：ESLint 规则 `@angular-eslint/component-class-suffix`（允许后缀 `Directive / Component / Base / Widget`）。

### 2.3 组件选择器命名

- 选择器使用 **kebab-case**，项目默认前缀为 **`app-`**（`angular.json` 中 `prefix: "app"`）。

```ts
@Component({
  selector: 'app-form-field'
})
```

- **现有前缀例外（仅限存量布局部件，新增组件不再沿用）**：
  - 主框架组件：`layout-basic`、`layout-blank`、`layout-passport`
  - 顶栏部件：`header-user`、`header-notify`、`header-i18n`、`header-fullscreen`、`header-ai-chat`
  - JSON Schema 示例部件：`test`
- **【建议】** 新增组件一律使用 `app-` 前缀，保持与脚手架默认前缀一致；不要为某个新模块发明新前缀。

依据：`angular.json` 的 `prefix: "app"`；ESLint 中 `@angular-eslint/component-selector` 配置（`prefix: ['app', 'test']`、`style: 'kebab-case'`，当前为关闭状态，按约定执行）。

### 2.4 输入、输出、属性命名

- `@Input` / `@Output` 属性使用 **camelCase**。
- 输出事件使用 **动词或动词短语**，描述“发生了什么”，如 `selectedChanged`、`valueChange`、`cleared`、`apply`、`refresh`。
- 需要双向绑定时，输入属性与输出事件成对出现，事件名 = 输入名 + `Change`：

```ts
@Input() value: any;
@Output() readonly valueChange = new EventEmitter<string>();
```

```html
<app-multi-func-input [(value)]="searchText"></app-multi-func-input>
```

依据：`MultiFuncInputComponent`、`SignalTimeRangeComponent`（`value/valueChange`、`refreshSeconds/refreshSecondsChange`）。

### 2.5 管道、服务命名

- 管道类名以 `Pipe` 结尾，管道名（`name`）使用 camelCase，如 `safe`、`elapsedTime`。
- 自定义管道必须为**纯管道** **【强制】**（ESLint `@angular-eslint/no-pipe-impure`）。
- 服务类名以 `Service` 结尾；全局单例使用 `@Injectable({ providedIn: 'root' })`，如 `AiChatModalService`。

---

## 3. 组件引入规范

### 3.1 模块式组件（存量主模式）

- 业务组件在所属业务模块的 `declarations` 中注册，如 `MonitorModule`、`AlertModule`、`SettingModule`。
- 组件只能在**一个**模块中声明；需要跨模块复用时，统一由 `SharedModule` 声明并导出。

```ts
// shared/shared.module.ts
const COMPONENTS = [
  MultiFuncInputComponent,
  ToolbarComponent,
  FormFieldComponent,
  MonitorSelectMenuComponent,
  MonitorSelectListComponent,
  LabelSelectorComponent,
  SqlEditorComponent
  // ...
];

@NgModule({
  declarations: [...COMPONENTS, ...DIRECTIVES, HelpMessageShowComponent],
  exports: [/* ... */, ...COMPONENTS, ...DIRECTIVES]
})
export class SharedModule {}
```

- 业务模块只需引入 `SharedModule` 即可使用全部公共组件，**禁止**在业务模块中重复声明公共组件。

### 3.2 第三方组件库引入

- **ng-zorro-antd**：公共常用模块集中在 `shared/shared-zorro.module.ts` 的 `SHARED_ZORRO_MODULES` 中聚合，随 `SharedModule` 全局可用；业务模块有特殊需要时再单独引入（如 `NzTimePickerModule`、`NzTransferModule` 在 `AlertModule` 中单独引入）。
- **@delon**：公共模块集中在 `shared/shared-delon.module.ts` 的 `SHARED_DELON_MODULES`（`PageHeaderModule / STModule / SEModule / SVModule / ResultModule`）；布局类模块（`LayoutDefaultModule`、`GlobalFooterModule`、`SettingDrawerModule`）在 `LayoutModule` 中引入。
- **按需引入、禁止整包引入**：使用 `ng-zorro-antd/xxx` 的模块路径，不引入库根入口。

```ts
import { NzTableModule } from 'ng-zorro-antd/table';
import { NzModalModule } from 'ng-zorro-antd/modal';
```

- 【建议】只引入模板中实际使用的模块；未使用的模块不放入聚合数组。

### 3.3 独立组件（Standalone，新增组件推荐）

- 新增的、可独立交付的页面或组件优先使用 `standalone: true`，依赖直接写在 `imports` 中：

```ts
@Component({
  selector: 'app-signal-time-range',
  standalone: true,
  imports: [FormsModule, I18nPipe, SharedModule, NzDatePickerModule, NzSelectModule],
  template: ` ... `
})
export class SignalTimeRangeComponent {}
```

- standalone 组件需要公共组件时直接引入 `SharedModule` 或具体组件，不新建只含一个组件的模块。
- 参考：`SignalWorkspaceComponent`、`SignalNavigationComponent`、`SignalTimeRangeComponent`、`SignalStorageGuideComponent`、`LogStreamComponent`、`LogManageComponent`、`MetricsManageComponent`、`TraceManageComponent`、`LogIntegrationComponent`、`AlertIntegrationComponent`。

### 3.4 路由与懒加载

- 业务模块通过 `loadChildren` 懒加载；独立页面通过 `loadComponent` 懒加载：

```ts
{
  path: 'monitors',
  loadChildren: () => import('./monitor/monitor.module').then(m => m.MonitorModule)
},
{
  path: 'metrics/manage',
  loadComponent: () => import('./metrics/metrics-manage.component').then(m => m.MetricsManageComponent)
}
```

- 【建议】除仪表盘、登录等首屏必需页面外，业务页面一律懒加载，不进入主包。

### 3.5 import 语句组织 **【强制】**

ESLint `import/order` 规定的分组与顺序：

1. 第三方依赖（external）
2. 项目内部别名（internal，如 `@shared`、`@core`、`@env`）
3. 父级 / 同级相对路径（parent / sibling / index）

要求：组内按字母升序，组与组之间空一行；禁止重复导入同一模块（`import/no-duplicates`）。

```ts
// 正确示例
import { Component, Input } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

import { I18NService } from '@core';

import { MonitorService } from '../../../service/monitor.service';
```

### 3.6 动态组件

- 弹窗类动态组件通过服务配合 `NzModalService.create({ nzContent })` 创建，不在模板中写死，例如 `ChatComponent` 由 `AiChatModalService` 统一打开/关闭。
- 需要向动态组件传入初始值时，通过返回的 `componentInstance` 赋值；组件内必须考虑该赋值晚于 `ngOnInit` 的时序问题。

---

## 4. 参数（输入 / 输出 / 插槽）规范

### 4.1 输入参数 **【建议】**

- 对外暴露的可配置项一律使用 `@Input()`，并为其声明类型；无合理默认值的必填输入不做默认初始化。
- 需要拦截赋值时使用 getter / setter，而不是在 `ngOnChanges` 中散落副作用：

```ts
@Input()
get monitorId(): number {
  return this._monitorId;
}
set monitorId(monitorId: number) {
  this._monitorId = monitorId;
}
```

依据：`MonitorDataChartComponent.monitorId`。

- 保持输入属性名与模板绑定名一致（项目未启用 `@Input` 重命名限制，故约定上不改名、不使用 `@Input('alias')`）。
- 输入只用于“接收数据”，组件内部不得直接修改输入引用的对象内容后回传给父级。

### 4.2 输出事件 **【强制】**

- 输出统一声明为 `readonly` 的 `EventEmitter`（ESLint `@angular-eslint/prefer-output-readonly`）：

```ts
@Output() readonly selectedChanged = new EventEmitter<string>();
@Output() readonly cleared = new EventEmitter<void>();
```

- 事件只在用户动作或状态真实变化时触发，载荷使用强类型；无载荷的事件使用 `EventEmitter<void>`。
- 【建议】只声明实际会被触发的输出；废弃的输出及时删除，避免调用方监听永不触发的事件。

### 4.3 内容投影（插槽）

- 容器型组件通过 `@ContentChild` 收集具名模板，命名与模板引用变量一一对应：

```ts
@ContentChild('left', { static: true }) leftTemplateRef: TemplateRef<any> | undefined;
@ContentChild('right', { static: true }) rightTemplateRef: TemplateRef<any> | undefined;
```

- 简单内容投影使用默认 `<ng-content>`；存在多个投影区时使用 `select` 属性区分（参考 `SignalWorkspaceComponent` 的 `[workspace-actions]`、`[workspace-filters]`、`[workspace-results]`）。

### 4.4 表单组件接入规范

需要放入 Angular 表单的自定义输入组件，**必须**实现 `ControlValueAccessor`，并通过 `NG_VALUE_ACCESSOR` 注册，使其可直接使用 `[(ngModel)]` / `formControlName`：

```ts
@Component({
  selector: 'app-multi-func-input',
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => MultiFuncInputComponent),
      multi: true
    }
  ]
})
export class MultiFuncInputComponent implements ControlValueAccessor {
  // writeValue / registerOnChange / registerOnTouched
}
```

- 需要参与表单校验的组件，额外实现 `Validator` 并注册 `NG_VALIDATORS`（参考 `FormFieldComponent`、`ConfigurableFieldComponent`）。
- 【建议】组件内部需要维护“是否校验错误”状态时，以专门的状态字段（如 `validateStatus`）暴露，校验逻辑由内部子组件自洽完成。
- 适用组件：`MultiFuncInputComponent`、`FormFieldComponent`、`ConfigurableFieldComponent`、`LabelSelectorComponent`、`SqlEditorComponent`。

### 4.5 字段驱动型组件约定

- `FormFieldComponent` 是项目“后端字段定义（ParamDefine）→ UI 控件”的唯一分发入口，新字段类型在此组件的模板中扩展，不要在业务页面绕过它自行 `switch` 字段类型。
- 字段定义通过 `[item]` 传入，附加数据通过 `[extra]` 传入（如采集器列表、标签键、时间间隔类型）。
- 新旧数据结构兼容转换（如 `key-value`、`labels` 的对象与数组互转）收敛在该组件内部，业务页面只处理最终值。

---

## 5. 样式与样式隔离规范

### 5.1 样式文件组织

- 组件样式统一使用 **LESS**（`angular.json` 中 `@schematics/angular:component` 的 `style: "less"`），文件与组件同名：`.component.less`。
- 模板外置组件使用 `styleUrls`；独立组件使用内联 `styles` 或 `styleUrl`，与现有同类组件保持一致。
- 全局样式只允许放在 `src/styles.less` 及全局样式目录；组件默认不写全局选择器。

### 5.2 样式作用域

- **默认使用 Angular Emulated 隔离**，不显式声明 `encapsulation`。全工程仅 `SignalWorkspaceComponent` 因作为通用外壳使用 `ViewEncapsulation.None`。
- 确需关闭隔离时，必须同时满足：
  1. 组件为纯布局/外壳，不含可重入的业务页面；
  2. 所有样式规则以组件根 class 为父选择器，防止泄漏到全局。

```ts
@Component({
  selector: 'app-signal-workspace',
  encapsulation: ViewEncapsulation.None
})
```

### 5.3 穿透第三方组件样式

- 需要覆盖 ng-zorro 等第三方组件内部样式时，使用 `:host ::ng-deep`，**必须**挂在 `:host` 之下，禁止顶层裸用 `::ng-deep`：

```less
// 正确：穿透范围被限制在本组件内
:host ::ng-deep {
  .monaco-editor .suggest-widget {
    z-index: 9999 !important;
  }
}

// 错误：脱离组件作用域，样式会泄漏到全局
::ng-deep .ant-card-body {
  padding: 0;
}
```

- 【建议】穿透前先确认第三方组件是否提供了输入属性或主题变量可解决，能用配置解决的不用覆盖样式。
- Stylelint 已将 `ng-deep` 列为允许的伪元素，并允许 `nz-`、`app-`、`g2-` 开头的元素选择器。

### 5.4 样式作用域锚点 **【建议】**

- 组件内自定义样式统一以**组件根容器 class**（如 `.signal-workspace`、`.chat-container`）或 `:host` 作为父选择器，不依赖纯标签选择器。
- 页面内部专用区域（如弹窗表单）建立专用容器 class（如 `.modal-form-container`），所有规则挂在该 class 下，保证模板结构调整时样式仍然可控。

```less
:host {
  display: block;
}

.chat-container {
  .message-item {
    // ...
  }
}
```

### 5.5 深色主题

- 深色主题通过 `body`/根节点上的 `data-theme` 属性切换，组件内使用以下两种既有方式适配，不自行新增主题开关机制：

```less
[data-theme='dark'] :host,
.dark :host {
  // 深色样式
}
```

```less
:host-context([data-theme='dark']) .chat-container {
  // 深色样式
}
```

依据：`HelpMessageShowComponent`、`LayoutBasicComponent`、`ChatComponent` 的深色主题写法。

### 5.6 LESS 书写规则 **【强制】**

- Stylelint 采用 `stylelint-config-standard` + `stylelint-config-rational-order`，属性声明按**合理顺序**排列（定位 → 盒模型 → 文字 → 视觉 → 其他），提交前自动校验。
- 启用 `declaration-block-no-ignored-properties`，禁止书写被相邻属性抵消的无效声明。
- LESS 中使用 `@import` 时以字符串形式书写（`import-notation: "string"`）。

---

## 6. 配套代码风格（Prettier / ESLint 要点）

| 项 | 规则 |
| -- | -- |
| 缩进 | 2 个空格，禁止 Tab |
| 引号 | 单引号（TS / JS） |
| 行宽 | 140 字符 |
| 分号 | 语句末尾必须加分号 |
| 尾逗号 | 不加 |
| 箭头函数参数 | 单参数省略括号（`arrowParens: 'avoid'`） |
| 换行符 | LF |
| 字符串拼接 | 优先模板字符串（`prefer-template`） |
| 对象合并 | 优先展开语法（`prefer-object-spread`） |
| 数组类型 | 使用 `T[]` 简单数组写法（`array-type: array-simple`） |
| `@Attribute()` | 禁止使用属性参数装饰器（`no-attribute-decorator`） |

---

## 7. 提交前检查清单

新增或修改组件时，逐项确认：

- [ ] 文件名 / 目录名为 kebab-case，类型后缀正确（`.component.ts` 等）
- [ ] 组件类名为 PascalCase，并以 `Component / Directive / Base / Widget` 结尾
- [ ] 选择器使用 `app-` 前缀（存量 `layout-` / `header-` 前缀仅限旧部件）
- [ ] 组件在正确的位置注册：业务模块 `declarations` / `SharedModule` 导出 / standalone `imports`
- [ ] ng-zorro、@delon 模块按需引入，无未使用的导入
- [ ] 输入有明确类型；输出为 `readonly EventEmitter` 且确实会被触发
- [ ] 表单组件已实现 `ControlValueAccessor`，需校验时实现 `Validator`
- [ ] 容器组件的投影插槽与 `@ContentChild` 命名一致
- [ ] 样式使用 LESS，默认 Emulated 隔离；穿透样式均为 `:host ::ng-deep`
- [ ] 自定义规则均挂在根容器 class 或 `:host` 之下
- [ ] 深色主题通过 `data-theme` / `:host-context` 适配
- [ ] ESLint、Prettier、Stylelint 全部通过，无 `debugger`、注释掉的死代码

---

## 8. 依据文件

本规范的所有强制规则均可追溯至下列配置文件与组件实现：

| 依据 | 文件 |
| -- | -- |
| 组件前缀、样式语言、严格模式 | `web-app/angular.json` |
| 组件后缀、输出只读、纯管道、import 顺序等 | `web-app/.eslintrc.js` |
| 代码格式 | `web-app/.prettierrc.js` |
| LESS 校验、属性顺序、`ng-deep` 白名单 | `web-app/.stylelintrc` |
| 编辑器基础约定 | `web-app/.editorconfig` |
| 公共组件注册与导出 | `web-app/src/app/shared/shared.module.ts` |
| ng-zorro 模块聚合 | `web-app/src/app/shared/shared-zorro.module.ts` |
| @delon 模块聚合 | `web-app/src/app/shared/shared-delon.module.ts` |
| 表单组件 CVA 范例 | `web-app/src/app/shared/components/multi-func-input/multi-func-input.component.ts` |
| 校验器范例 | `web-app/src/app/shared/components/form-field/form-field.component.ts` |
| 关闭样式隔离的特例 | `web-app/src/app/routes/observability/signal-workspace.component.ts` |
| `:host ::ng-deep` 范例 | `web-app/src/app/shared/components/sql-editor/sql-editor.component.less` |
| 动态组件范例 | `web-app/src/app/shared/services/ai-chat-modal.service.ts` |
