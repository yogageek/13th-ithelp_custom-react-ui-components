# 哎呀！不小心刻了一套 React UI 元件庫 — 教學總覽

> 整理自 [2021 iThome 鐵人賽](https://ithelp.ithome.com.tw/users/20111490/ironman/3999) 及 [Storybook](https://timingjl.github.io/13th-ithelp_custom-react-ui-components/)，適合搭配 Obsidian 複習使用。

---

## 目錄

- [[#專案架構]]
- [[#主題 Theme]]
- [[#Hooks]]
  - [[#useColor]]
  - [[#usePagination]]
- [[#工具函式 Utils]]
- [[#基礎元件]]
  - [[#Portal]]
  - [[#Option]]
  - [[#Icons — FaSpinner]]
- [[#數據輸入元件]]
  - [[#Button]]
  - [[#Checkbox]]
  - [[#Radio / RadioGroup]]
  - [[#Rate]]
  - [[#Select]]
  - [[#Slider / CustomSlider]]
  - [[#Switch]]
  - [[#TextField]]
  - [[#Upload]]
- [[#數據展示元件]]
  - [[#Accordion]]
  - [[#Badge]]
  - [[#Card / Card.Meta]]
  - [[#Carousel]]
  - [[#Chip]]
  - [[#InfiniteScroll]]
  - [[#Skeleton]]
  - [[#Table]]
  - [[#Tooltip]]
- [[#導航元件]]
  - [[#Breadcrumb]]
  - [[#Drawer]]
  - [[#Dropdown]]
  - [[#Pagination]]
  - [[#Tabs]]
- [[#反饋元件]]
  - [[#FormControl]]
  - [[#Modal]]
  - [[#Dialog]]
  - [[#ProgressBar]]
  - [[#ProgressCircle]]
  - [[#Spin]]
  - [[#Toast]]

---

## 專案架構

```
src/
├── components/     # 所有 UI 元件
├── hooks/          # 自訂 Hooks（useColor、usePagination）
├── theme/          # 主題色彩設定
├── utils/          # 工具函式
└── stories/        # Storybook 範例
```

技術棧：
- **React**（函式元件 + Hooks）
- **styled-components**（CSS-in-JS 樣式）
- **PropTypes**（型別檢查）
- **Material-UI Icons**（圖示庫）

---

## 主題 Theme

**路徑**：`src/theme/`

```js
const defaultTheme = {
  color: {
    primary:  '#1976d2',
    secondary: '#dc004e',
    disable:  '#dadada',
    error:    '#d0021b',
  },
};
```

- 所有元件透過 `styled-components` 的 `ThemeProvider` 取用主題色。
- `themeColor` 接受 `'primary'`、`'secondary'` 或任意色票字串（例如 `'#FF5733'`）。

---

## Hooks

### useColor

**路徑**：`src/hooks/useColor.jsx`

```jsx
import { useTheme } from 'styled-components';

export const useColor = () => {
  const theme = useTheme();

  const makeColor = ({ themeColor, isDisabled }) => {
    const madeColor = theme.color[themeColor] || themeColor;
    return isDisabled ? theme.color.disable : madeColor;
  };

  return { makeColor };
};
```

- 從 styled-components ThemeProvider 取得 theme。
- `makeColor` 依據 `themeColor` 與 `isDisabled` 決定最終色彩。
- 若 `themeColor` 是 `'primary'`／`'secondary'` 則使用主題色；否則直接使用傳入的色碼。

---

### usePagination

**路徑**：`src/hooks/usePagination.jsx`

```jsx
export const usePagination = ({ page, pageSize, total, withEllipsis, onChange }) => {
  const totalPage = Math.ceil(total / pageSize);

  // 建立頁碼 items 陣列
  const items = [...Array(totalPage).keys()].map((key) => ({
    type: 'page',
    isCurrent: page === key + 1,
    page: key + 1,
    onClick: () => onChange(key + 1),
  }));

  // withEllipsis 模式：距離目前頁碼超過 1 的頁碼以省略符號代替
  const ellipsisItems = /* ... */;

  const handleClickNext = () => onChange(Math.min(page + 1, totalPage));
  const handleClickPrev = () => onChange(Math.max(page - 1, 1));

  return {
    items: withEllipsis ? ellipsisItems : items,
    totalPage,
    handleClickNext,
    handleClickPrev,
  };
};
```

- 回傳 `items`：每一筆物件包含 `type`、`isCurrent`、`page`、`onClick`。
- `withEllipsis`：頁碼過多時以 `start-ellipsis` / `end-ellipsis` type 取代。

---

## 工具函式 Utils

**路徑**：`src/utils/event.js`

```js
export const findAttributeInEvent = (event, attr) => {
  const end = event.currentTarget;
  let temp = event.target;
  let dataId = temp.getAttribute(attr);

  while (temp !== end && !dataId) {
    temp = temp.parentElement;
    if (temp === null) break;
    dataId = temp.getAttribute(attr);
  }
  return dataId;
};
```

- 向上遍歷 DOM 樹，找到指定的 `data-*` 屬性值。
- 常用於點擊事件中判斷點擊目標（例如 Dropdown 的 `data-dropdown-id`）。

---

## 基礎元件

### Portal

**路徑**：`src/components/Portal/index.jsx`

**說明**：將子元件渲染到指定的 DOM 節點（預設 `#portal-root`），脫離原本的 DOM 層級，常用於 Modal、Drawer、Tooltip、Dropdown 等需要置頂的元件。

```jsx
import ReactDOM from 'react-dom';

const Portal = ({ children, customRootId }) => {
  const rootId = customRootId || 'portal-root';
  // 若不存在則建立 div 掛到 body
  let portalRoot = document.getElementById(rootId) || (() => {
    const div = document.createElement('div');
    div.id = rootId;
    document.body.appendChild(div);
    return div;
  })();

  // 元件 unmount 時移除 portalRoot
  useEffect(() => () => portalRoot?.parentElement?.removeChild(portalRoot), [portalRoot]);

  return ReactDOM.createPortal(children, portalRoot);
};
```

**重點**：
- `ReactDOM.createPortal(children, container)` — React 官方 Portal API。
- 視覺上仍遵循 z-index 堆疊規則，邏輯上仍屬於 React 元件樹。

---

### Option

**路徑**：`src/components/Option/index.jsx`

**說明**：底層選取元件，提供勾選／未勾選圖示、禁用狀態，被 `Checkbox` 與 `Radio` 繼承使用。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `isChecked` | `bool` | `false` | 是否被勾選 |
| `isDisabled` | `bool` | `false` | 是否禁用 |
| `themeColor` | `string` | `'primary'` | 主題色 |
| `checkedIcon` | `element` | `<CheckBoxIcon />` | 選中圖示 |
| `unCheckedIcon` | `element` | `<CheckBoxOutlineBlankIcon />` | 未選中圖示 |
| `onClick` | `func` | `() => {}` | 點擊事件 |
| `children` | `string/element` | `''` | 標籤文字 |

---

### Icons — FaSpinner

**路徑**：`src/components/Icons/FaSpinner.jsx`

自訂 SVG spinner 圖示，可搭配 CSS `animation: rotate` 使用：

```jsx
import { FaSpinner } from 'components/Icons/FaSpinner';

// 旋轉動畫範例
const RotateContainer = styled.div`
  width: 40px; height: 40px;
  animation: ${rotateAnimation} 1000ms ease-in-out infinite;
`;

<RotateContainer><FaSpinner /></RotateContainer>
```

---

## 數據輸入元件

### Button

**路徑**：`src/components/Button/index.jsx`

**說明**：可點擊按鈕，觸發相對應業務邏輯，支援多種樣式變體、Loading 狀態及前後圖示。

#### Props

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `variant` | `'contained'｜'outlined'｜'text'` | `'contained'` | 按鈕樣式 |
| `themeColor` | `string` | `'primary'` | 主題色 |
| `isLoading` | `bool` | `false` | 載入中狀態（顯示 spinner） |
| `isDisabled` | `bool` | `false` | 禁用狀態 |
| `startIcon` | `element` | `null` | 左方圖示 |
| `endIcon` | `element` | `null` | 右方圖示 |
| `onClick` | `func` | `() => {}` | 點擊事件 |
| `children` | `string/element` | **必填** | 按鈕文字 |

#### 樣式邏輯

```jsx
const variantMap = {
  contained: css`background: ${btnColor}; color: #FFF;`,
  outlined:  css`background: #FFF; color: ${btnColor}; border: 1px solid ${btnColor};`,
  text:      css`background: #FFF; color: ${btnColor};`,
};
```

#### 使用範例

```jsx
// 基本按鈕
<Button>Button</Button>

// outlined 樣式
<Button variant="outlined">Button</Button>

// 載入中
<Button isLoading>Button</Button>

// 禁用
<Button isDisabled>Button</Button>

// 帶圖示
<Button startIcon={<CloudDownloadIcon />}>Button</Button>

// 自訂樣式（漸層 + 圓角）
<Button
  endIcon={<CloudDownloadIcon />}
  style={{ background: 'linear-gradient(45deg, #FE6B8B 30%, #FF8E53 90%)', borderRadius: 50 }}
>
  Button
</Button>
```

---

### Checkbox

**路徑**：`src/components/Checkbox/index.jsx`

**說明**：多選框元件，底層使用 `Option`，適合群組多項選擇。

```jsx
// Checkbox 只是 Option 的薄包裝
const Checkbox = (props) => <Option {...props} />;
```

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `isChecked` | `bool` | `false` | 是否勾選 |
| `isDisabled` | `bool` | `false` | 禁用 |
| `themeColor` | `string` | `'primary'` | 主題色 |
| `onClick` | `func` | `() => {}` | 點擊事件 |
| `children` | `string/element` | `''` | 標籤文字 |

#### 使用範例

```jsx
const [isChecked, setIsChecked] = useState(false);

<Checkbox
  isChecked={isChecked}
  onClick={() => setIsChecked(prev => !prev)}
>
  Checkbox
</Checkbox>

// 禁用
<Checkbox isDisabled>Checkbox</Checkbox>
```

---

### Radio / RadioGroup

**路徑**：`src/components/Radio/`

**說明**：單選框元件，使用 `RadioButtonCheckedIcon` / `RadioButtonUncheckedIcon` 作為圖示，適合選項不多且希望用戶一次看到所有選項的情境。

> 若選項多到需要折疊，建議改用 `Select` 下拉選單。

#### Radio Props

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `isChecked` | `bool` | `false` | 是否選中 |
| `isDisabled` | `bool` | `false` | 禁用 |
| `themeColor` | `string` | `'primary'` | 主題色 |
| `value` | `string` | `null` | 在 RadioGroup 中標識此選項 |
| `onClick` | `func` | `() => {}` | 點擊事件 |
| `children` | `string/element` | `''` | 標籤文字 |

#### RadioGroup Props

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `value` | `string/number` | `null` | 目前選中的值 |
| `onChange` | `func` | `() => {}` | 值改變時的 callback |
| `columns` | `number` | `1` | 欄數（Grid layout） |
| `children` | `element/array` | `null` | Radio 子元件 |

#### 使用範例

```jsx
// 獨立使用
const [isChecked, setIsChecked] = useState(false);
<Radio isChecked={isChecked} onClick={() => setIsChecked(true)}>Radio</Radio>

// 搭配 RadioGroup
const [selectedValue, setSelectedValue] = useState('');
<RadioGroup value={selectedValue} onChange={setSelectedValue} columns={2}>
  <Radio value="male">Male</Radio>
  <Radio value="female">Female</Radio>
  <Radio value="others">Others</Radio>
</RadioGroup>
```

---

### Rate

**路徑**：`src/components/Rate/index.jsx`

**說明**：評分元件，支援半顆星、自訂字符、禁用互動等功能。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `count` | `number` | `5` | 星星總數 |
| `defaultValue` | `number` | `0` | 預設值 |
| `character` | `string/element` | `<StarIcon />` | 評分字符 |
| `size` | `number` | `32` | 字符大小（px） |
| `allowHalf` | `bool` | `false` | 是否允許半顆 |
| `isDisabled` | `bool` | `false` | 禁用互動 |
| `themeColor` | `string` | `'#FBDB14'` | 主題色 |
| `onChange` | `func` | `() => {}` | 值改變 callback |

#### 核心邏輯

- 每個字符由 `CharacterFirst`（左半，width 50%，overflow hidden）和 `CharacterSecond`（右半）組成。
- `allowHalf` 模式下 `CharacterFirst` 才顯示，用 `itemKey + 0.5` 代表半顆。
- `previewValue`（hover 預覽值）與 `innerValue`（實際值）分開管理。

#### 使用範例

```jsx
// 基本
<Rate defaultValue={3} onChange={(v) => console.log(v)} />

// 半顆星
<Rate defaultValue={3.5} allowHalf onChange={(v) => console.log(v)} />

// 自訂字符
<Rate character={<FavoriteBorderIcon />} allowHalf defaultValue={2.5} />
<Rate character="好" defaultValue={3} />

// 禁用（只展示）
<Rate defaultValue={4} isDisabled />
```

---

### Select

**路徑**：`src/components/Select/index.jsx`

**說明**：下拉選擇器，基於 `Dropdown` 實作，觸發時彈出選單。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `options` | `array` | `[]` | 選項（含 `value`、`label`） |
| `value` | `string` | `''` | 目前選中的值 |
| `placeholder` | `string` | `''` | 未選擇時的提示文字 |
| `isDisabled` | `bool` | `false` | 禁用 |
| `isLoading` | `bool` | `false` | 載入中 |
| `onSelect` | `func` | `() => {}` | 選中時 callback |

#### 使用範例

```jsx
const options = [
  { value: 'AZ', label: 'AZ 疫苗' },
  { value: 'BNT', label: 'BNT 疫苗' },
  { value: 'Moderna', label: '莫德納疫苗' },
];

const [selectedValue, setSelectedValue] = useState('');

<Select
  options={options}
  value={selectedValue}
  placeholder="請選擇疫苗"
  onSelect={(value) => setSelectedValue(value)}
/>

// 禁用
<Select options={options} isDisabled />

// 載入中
<Select options={options} isLoading />
```

---

### Slider / CustomSlider

**路徑**：`src/components/Slider/`

**說明**：滑動型輸入器，允許在數值區間內選擇連續或離散值。

#### Slider (input[type=range] 封裝)

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `min` | `number` | `0` | 最小值 |
| `max` | `number` | `100` | 最大值 |
| `step` | `number` | `1` | 步長（需能整除 max-min） |
| `defaultValue` | `number` | `0` | 預設值 |
| `themeColor` | `string` | `'primary'` | 主題色 |
| `onChange` | `func` | `() => {}` | 值改變 callback |

```jsx
// 透過 CSS `before` 偽元素渲染 track（已滑過的部分）
&[type='range']:before {
  content: '';
  position: absolute;
  z-index: -1;
  width: ${widthRatio}%;
  background: ${color};
}
```

#### CustomSlider (純手刻，使用 RxJS)

使用 `rxjs` 的 `fromEvent` + `concatMap` + `takeUntil` 處理拖曳事件。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `min` | `number` | `0` | 最小值 |
| `max` | `number` | `100` | 最大值 |
| `defaultValue` | `number` | `0` | 預設值 |
| `onChange` | `func` | `() => {}` | 值改變 callback |

```jsx
// RxJS 拖曳事件處理
mouseDown.pipe(
  concatMap(() => mouseMove.pipe(takeUntil(mouseUp))),
  map((moveEvent) => moveEvent.clientX),
).subscribe((mousePosX) => {
  handleUpdatePosition({ mousePosX });
});
```

#### 使用範例

```jsx
// 基本
const [value, setValue] = useState(0);
<Slider onChange={(e) => setValue(e.target.value)} />

// 步長
<Slider min={0} max={8} step={2} />

// 自訂顏色
<Slider defaultValue={50} themeColor="#42f5c5" />

// 純手刻版
<CustomSlider defaultValue={50} onChange={(v) => console.log(v)} />
```

---

### Switch

**路徑**：`src/components/Switch/index.jsx`

**說明**：開關選擇器，觸發時立即改變狀態（不同於 Checkbox 需提交後生效）。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `isChecked` | `bool` | `null` | 開啟狀態 |
| `isDisabled` | `bool` | `false` | 禁用 |
| `themeColor` | `string` | `'primary'` | 主題色 |
| `size` | `'default'｜'small'` | `'default'` | 大小 |
| `checkedChildren` | `string` | `''` | 開啟時的文字 |
| `unCheckedChildren` | `string` | `''` | 關閉時的文字 |
| `onChange` | `func` | `() => {}` | 狀態改變 callback |

#### 核心邏輯

- `thumbSize`：`size === 'small'` 時為 12px，預設 18px。
- `switchWidth = thumbSize + labelWidth`，長度隨文字自適應。
- `useLayoutEffect` 量測 label 寬度以計算 switchWidth。

#### 使用範例

```jsx
const [isChecked, setIsChecked] = useState(false);

// 基本
<Switch isChecked={isChecked} onChange={() => setIsChecked(prev => !prev)} />

// 帶文字
<Switch
  isChecked={isChecked}
  checkedChildren="開啟"
  unCheckedChildren="關閉"
  onChange={() => setIsChecked(prev => !prev)}
/>

// 禁用
<Switch isChecked={false} isDisabled />
<Switch isChecked={true}  isDisabled />

// 小尺寸
<Switch size="small" isChecked={isChecked} onChange={() => setIsChecked(prev => !prev)} />
```

---

### TextField

**路徑**：`src/components/TextField/index.jsx`

**說明**：文字輸入框，支援前後綴、錯誤狀態及禁用狀態。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `prefix` | `element` | `null` | 前綴元件 |
| `suffix` | `element` | `null` | 後綴元件 |
| `placeholder` | `string` | `''` | 佔位文字 |
| `isError` | `bool` | `false` | 錯誤樣式 |
| `isDisabled` | `bool` | `false` | 禁用 |
| `onChange` | `func` | `() => {}` | 值改變 callback |

#### 使用範例

```jsx
// 基本
<TextField placeholder="Text Field" />

// 前綴
<TextField prefix={<InputAdornment position="start">$</InputAdornment>} placeholder="請輸入金額" />

// 後綴
<TextField suffix={<SearchIcon />} placeholder="搜尋" />

// 全寬（透過 styled 覆寫）
const FullWidthTextField = styled(TextField)`width: 100%;`;
<FullWidthTextField placeholder="Full width" />

// 錯誤
<TextField isError placeholder="Error Text Field" />

// 禁用
<TextField isDisabled placeholder="Disabled Text Field" />
```

---

### Upload

**路徑**：`src/components/Upload/index.jsx`

**說明**：上傳元件，透過隱藏的 `<input type="file">` 觸發檔案選擇。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `resetKey` | `number` | `0` | 鍵值改變時重設 input（清空已選檔案） |
| `accept` | `string` | `undefined` | 限制檔案類型（如 `'image/*'`） |
| `multiple` | `bool` | `false` | 是否可多選 |
| `onChange` | `func` | `() => {}` | 選取檔案時 callback，傳入 `FileList` |
| `children` | `element` | **必填** | 觸發上傳的按鈕元件 |

#### 核心邏輯

```jsx
const handleOnClickUpload = () => inputFileRef.current.click();

// 子元件的 onClick 被 cloneElement 替換
React.cloneElement(children, { onClick: handleOnClickUpload })
```

#### 使用範例

```jsx
// 基本上傳
const [uploadFile, setUploadFile] = useState(null);
const [resetKey, setResetKey] = useState(0);

<Upload resetKey={resetKey} onChange={(files) => setUploadFile(files[0])}>
  <Button startIcon={<CloudUploadIcon />}>上傳</Button>
</Upload>

// 清空重設
<Button onClick={() => { setResetKey(prev => prev + 1); setUploadFile(null); }}>重設</Button>

// 圖片預覽（FileReader）
<Upload accept="image/*" onChange={(files) => {
  const reader = new FileReader();
  reader.addEventListener('load', () => setImageSrc(reader.result));
  reader.readAsDataURL(files[0]);
}}>
  <Button>上傳圖片</Button>
</Upload>

// 多選
<Upload accept="image/*" multiple onChange={(files) => { /* files 是 FileList */ }}>
  <Button>上傳多張圖片</Button>
</Upload>
```

---

## 數據展示元件

### Accordion

**路徑**：`src/components/Accordion/`

**說明**：可折疊／展開的內容區域元件，適合顯示內容複雜或龐大的頁面，透過分區塊顯示及隱藏來改善體驗。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `header` | `string/element` | **必填** | 標題內容 |
| `children` | `string/element` | **必填** | 可折疊的 panel 內容 |
| `isExpand` | `bool` | `false` | 是否展開 |
| `onClick` | `func` | `() => {}` | 標題點擊事件 |

#### 核心邏輯

- **Header**：使用 rotate CSS 動畫讓箭頭旋轉 180 度表示展開／收合狀態。
- **Panel**：使用 `max-height` + `overflow: hidden` + `transition` 實作收合動畫，`max-height` 由 `ref.current.scrollHeight` 動態決定。

```jsx
// Panel 動畫
const StyledPanel = styled.div`
  max-height: ${(props) => props.$maxHeight}px;
  overflow: hidden;
  transition: max-height 0.3s cubic-bezier(0.4, 0, 0.2, 1);
`;

// isExpand 決定 max-height
<StyledPanel $maxHeight={isExpand ? panelRef.current?.scrollHeight : 0}>
  {panel}
</StyledPanel>
```

#### 使用範例

```jsx
// 基本單個
const [isExpand, setIsExpand] = useState(false);
<Accordion
  header="header"
  isExpand={isExpand}
  onClick={() => setIsExpand(prev => !prev)}
>
  Panel 內容
</Accordion>

// 手風琴群組（每次只展開一個）
const [activeKey, setActiveKey] = useState(null);
{[0,1,2,3].map(key => (
  <Accordion
    key={key}
    header={`header__${key + 1}`}
    isExpand={activeKey === key}
    onClick={() => setActiveKey(activeKey === key ? null : key)}
  >
    <div>Lorem Ipsum...</div>
  </Accordion>
))}
```

---

### Badge

**路徑**：`src/components/Badge/index.jsx`

**說明**：在子元件角落顯示小徽章，通常用來顯示未讀訊息數量等需要醒目提示的資訊。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `badgeContent` | `number` | `null` | 徽章數字 |
| `themeColor` | `string` | `'#F85149'` | 主題色 |
| `placement` | `'top-left'｜'top-right'｜'bottom-left'｜'bottom-right'` | `'top-right'` | 徽章位置 |
| `max` | `number` | `99` | 最大顯示值（超過顯示 `max+`） |
| `variant` | `'standard'｜'dot'` | `'standard'` | 變化模式 |
| `showZero` | `bool` | `false` | 是否顯示 0 |
| `children` | `element` | **必填** | 被包裹的元件 |

#### 使用範例

```jsx
<Badge badgeContent={7} children={<MailIcon />} />

// 位置
<Badge placement="top-left"    badgeContent={7}><MailIcon /></Badge>
<Badge placement="bottom-right" badgeContent={7}><MailIcon /></Badge>

// 最大值
<Badge max={87} badgeContent={120}><MailIcon /></Badge> // 顯示 "87+"

// 點狀
<Badge variant="dot" badgeContent={7}><MailIcon /></Badge>

// 顯示 0
<Badge showZero badgeContent={0}><MailIcon /></Badge>
```

---

### Card / Card.Meta

**路徑**：`src/components/Card/`

**說明**：顯示單個主題內容及操作的卡片，通常包含圖片、標題、描述或操作按鈕。

#### Card Props

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `cover` | `element` | `null` | 封面媒體（通常是 img） |
| `variant` | `'vertical'｜'horizontal'｜'horizontal-reverse'` | `'vertical'` | 排版方向 |
| `footer` | `element` | `null` | 卡片底部操作區 |
| `children` | `element` | `null` | 主要內容 |

#### Card.Meta Props

| Prop | 型別 | 說明 |
|---|---|---|
| `avatarUrl` | `string` | 頭像圖片 URL |
| `title` | `string` | 標題 |
| `description` | `string` | 描述 |

#### 使用範例

```jsx
<Card
  cover={<img src="https://example.com/image.jpg" alt="" style={{ objectFit: 'cover' }} />}
  footer={
    <div style={{ padding: 16 }}>
      <ThumbUpIcon /> <ShareIcon />
    </div>
  }
>
  <Card.Meta
    avatarUrl="https://example.com/avatar.png"
    title="2021 iThome 鐵人賽"
    description="喚醒心中最強大的鐵人"
  />
</Card>

// 橫向排版
<Card variant="horizontal" cover={...}>...</Card>
<Card variant="horizontal-reverse" cover={...}>...</Card>
```

---

### Carousel

**路徑**：`src/components/Carousel/index.jsx`

**說明**：輪播元件（旋轉木馬），適用於圖片或卡片的輪播展示。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `dataSource` | `string[]` | `[]` | 圖片 URL 陣列 |
| `hasDots` | `bool` | `true` | 是否顯示指示點 |
| `hasControlArrow` | `bool` | `true` | 是否顯示切換箭頭 |
| `autoplay` | `bool` | `false` | 是否自動播放（每 3 秒） |

#### 核心邏輯

- 每張圖片絕對定位，`left = (itemIndex - currentIndex) * imageWidth`，利用 `transition` 實現滑動動畫。
- 自動播放使用 `setInterval`，元件 unmount 或 `autoplay` 變更時 `clearInterval`。
- `useCallback` 確保 `handleClickNext` 依賴的 `currentIndex` 能正確取得。

```jsx
const makePosition = ({ itemIndex }) => (itemIndex - currentIndex) * imageWidth;

// 自動播放
useEffect(() => {
  let intervalId;
  if (autoplay) intervalId = setInterval(handleClickNext, 3000);
  return () => clearInterval(intervalId);
}, [autoplay, handleClickNext]);
```

#### 使用範例

```jsx
import birdImg from './assets/bird.jpeg';
import duckImg from './assets/duck.jpeg';

<Carousel
  dataSource={[birdImg, duckImg, eagleImg, frogImg]}
  hasDots
  hasControlArrow
  autoplay
/>
```

---

### Chip

**路徑**：`src/components/Chip/index.jsx`

**說明**：用於標記屬性、標籤或分類篩選的標籤元件。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `label` | `string/element` | **必填** | 內容 |
| `variant` | `'contained'｜'outlined'` | `'contained'` | 樣式 |
| `themeColor` | `string` | `'primary'` | 主題色 |
| `icon` | `element` | `null` | 前綴圖示 |
| `deleteIcon` | `element` | `null` | 自訂刪除圖示 |
| `onDelete` | `func｜null` | `null` | 刪除事件（傳入時顯示刪除圖示） |

#### 使用範例

```jsx
<Chip label="Chip" />
<Chip label="Outlined" variant="outlined" />

// 帶刪除功能
<Chip label="with onDelete" onDelete={() => null} />

// 帶前綴圖示
<Chip label="icon with onDelete" icon={<FaceIcon />} onDelete={() => null} />

// 自訂刪除圖示
<Chip label="custom deleteIcon" deleteIcon={<DoneIcon />} />
```

---

### InfiniteScroll

**路徑**：`src/components/InfiniteScroll/index.jsx`

**說明**：無限滾動載入元件，使用 `IntersectionObserver` 監聽底部載入指示器，當進入視窗時觸發下一頁載入。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `children` | `element[]` | **必填** | 列表項目 |
| `onScrollBottom` | `func` | `undefined` | 捲動到底部的 callback |

#### 核心邏輯

```jsx
// 使用 IntersectionObserver 監聽載入指示器
const intersectionObserver = new IntersectionObserver((entries) => {
  if (entries[0].isIntersecting) {
    onScrollBottom();
  }
});
intersectionObserver.observe(loadingRef.current, { threshold: 0.5 });
```

#### 使用範例

```jsx
const [dataSource, setDataSource] = useState([]);
const [page, setPage] = useState(1);
const [isLoading, setIsLoading] = useState(false);

useEffect(() => {
  setIsLoading(true);
  fetch(`https://picsum.photos/v2/list?page=${page}&limit=10`)
    .then(res => res.json())
    .then(data => {
      setDataSource(prev => [...prev, ...data]);
      setIsLoading(false);
    });
}, [page]);

<InfiniteScroll onScrollBottom={() => !isLoading && setPage(prev => prev + 1)}>
  {dataSource.map(({ id, author }) => (
    <div key={id}>{author}</div>
  ))}
</InfiniteScroll>
```

---

### Skeleton

**路徑**：`src/components/Skeleton/`

**說明**：骨架載入元件（Skeleton Screen Loading），在頁面載入完成前顯示頁面大致框架，載入後替換為真實資料。與 Spin 的差異：Skeleton 呈現頁面框架輪廓。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `variant` | `'colorBlock'｜'flash'｜'slide'` | `'slide'` | 動畫模式 |

#### 三種動畫模式

| 模式 | 效果 |
|---|---|
| `colorBlock` | 靜態灰色色塊，無動畫 |
| `flash` | 透明度呼吸閃爍（0.3 → 1 → 0.3） |
| `slide` | 從左至右滑過的光暈效果（使用 `::before` 偽元素） |

```jsx
// slide 動畫關鍵 CSS
const slide = keyframes`
  from { left: -150%; }
  to   { left: 100%;  }
`;

&:before {
  content: '';
  position: absolute;
  width: 80px;
  background: linear-gradient(to right, transparent 0%, #FFFFFF99 50%, transparent 100%);
  animation: ${slide} 1s cubic-bezier(0.4, 0.0, 0.2, 1) infinite;
}
```

#### 使用範例

```jsx
// 自訂尺寸（模擬頭像 + 文字行）
const Avatar   = (props) => <Skeleton style={{ width: 50, height: 50 }} {...props} />;
const TextLine = (props) => <Skeleton style={{ width: 300, height: 12 }} {...props} />;

<div style={{ display: 'flex', alignItems: 'center' }}>
  <Avatar variant="slide" />
  <div style={{ marginLeft: 16 }}>
    <TextLine variant="slide" />
    <TextLine variant="flash" style={{ width: 230, marginTop: 12 }} />
  </div>
</div>
```

---

### Table

**路徑**：`src/components/Table/index.jsx`

**說明**：表格元件，整齊顯示行列資料，支援固定欄（sticky column）及自訂渲染。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `columns` | `array` | `[]` | 欄位設定 |
| `dataSource` | `array` | `[]` | 資料來源 |

#### columns 設定格式

```js
const columns = [
  {
    title: 'Name',       // 欄位標題
    dataIndex: 'name',  // 對應 dataSource 的 key
    key: 'name',        // React key
    width: 130,         // 欄位寬度（px）
    fixed: true,        // 是否固定欄（sticky left）
    render: (value) => <Button>{value}</Button>, // 自訂渲染
  },
];
```

#### 使用範例

```jsx
const columns = [
  { title: 'Name', dataIndex: 'name', key: 'name', width: 130 },
  { title: 'Age',  dataIndex: 'age',  key: 'age',  width: 65  },
  { title: 'Address', dataIndex: 'address', key: 'address' },
];

const dataSource = [
  { key: '1', name: 'John Brown', age: 32, address: 'New York No.1' },
  { key: '2', name: 'Jim Green',  age: 42, address: 'London No.1'   },
];

<Table columns={columns} dataSource={dataSource} />

// Ant Design 風格（透過 styled 覆寫）
const AntdTable = styled(Table)`
  width: 100%;
  * { border: none; }
  td, th { padding: 16px; }
  tr { border-bottom: 1px solid #f0f0f0; }
`;

// 自訂 render
{
  title: '操作',
  dataIndex: 'actions',
  key: 'actions',
  render: () => <Button themeColor="secondary">刪除</Button>,
}
```

---

### Tooltip

**路徑**：`src/components/Tooltip/index.jsx`

**說明**：文字彈出提醒元件，hover 時顯示說明文字，支援 12 個方位及箭頭指示。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `content` | `string/element` | **必填** | 提示文字 |
| `children` | `string/element` | **必填** | 觸發提示的子元件 |
| `placement` | 見下表 | `'top'` | 出現位置 |
| `themeColor` | `string` | `'#101010'` | 背景色 |
| `showArrow` | `bool` | `true` | 是否顯示箭頭 |

#### placement 可選值

```
top-left      top         top-right
left-top                  right-top
left                      right
left-bottom               right-bottom
bottom-left   bottom      bottom-right
```

#### 核心邏輯

- 使用 `Portal` 渲染到 `#tooltip` 節點，避免被父元素 overflow 遮蓋。
- 透過 `getBoundingClientRect()` 取得子元件位置，計算 Tooltip 定位。
- `window.addEventListener('resize', handleOnResize)` 監聽視窗大小變化更新位置。

#### 使用範例

```jsx
<Tooltip content="喚醒心中最強大的鐵人">
  <InfoOutlinedIcon style={{ cursor: 'pointer' }} />
</Tooltip>

// 指定方位
<Tooltip content="提示文字" placement="bottom-left">
  <Button variant="outlined">Bottom Left</Button>
</Tooltip>

// 自訂顏色，無箭頭
<Tooltip content="提示" themeColor="#FE6B8B" showArrow={false}>
  <Button>Button</Button>
</Tooltip>
```

---

## 導航元件

### Breadcrumb

**路徑**：`src/components/Breadcrumb/`

**說明**：麵包屑導航元件，顯示當前系統層級位置，點擊可返回上層頁面。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `routes` | `array` | `[]` | 路由資訊陣列（含 `to`、`label`、`icon`） |
| `separator` | `string/element` | `<ArrowForwardIosIcon />` | 分隔符號 |
| `maxItems` | `number` | `8` | 最大顯示數量，超過則折疊 |

#### routes 格式

```js
const routes = [
  { to: '/home',    label: '首頁',   icon: <HomeIcon /> },
  { to: '/school',  label: '學校列表', icon: <SchoolIcon /> },
  { to: '/members', label: '會員列表' },
];
```

#### 折疊邏輯

當 `maxItems < routes.length` 時，只顯示第一項、`...`（點擊展開）、最後一項。

#### 使用範例

```jsx
<Breadcrumb routes={routes} />

// 自訂分隔符
<Breadcrumb separator="/" routes={routes} />

// 折疊
<Breadcrumb maxItems={2} routes={routes} />

// 完全自訂（使用 Breadcrumbs + Chip）
<Breadcrumbs>
  {routes.map(route => (
    <Chip key={route.label} label={route.label} icon={route.icon} />
  ))}
</Breadcrumbs>
```

---

### Drawer

**路徑**：`src/components/Drawer/index.jsx`

**說明**：抽屜元件，由螢幕邊緣滑出的浮動面板，常用於導航（Navigation Drawer）。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `isOpen` | `bool` | `false` | 是否顯示 |
| `placement` | `'top'｜'right'｜'bottom'｜'left'` | `'left'` | 滑出方向 |
| `onClose` | `func` | `() => {}` | 關閉事件（點擊遮罩觸發） |
| `animationDuration` | `number` | `200` | 動畫時間（ms） |
| `children` | `string/element` | **必填** | 抽屜內容 |

#### 核心邏輯

- 使用 `Portal` 渲染到頁面最上層。
- 使用 `keyframes` 定義四個方向的進出場動畫。
- `removeDOM` 狀態控制 DOM 的實際移除（等動畫播完後才移除）。

```jsx
// 關閉時先等動畫播完再移除 DOM
useEffect(() => {
  if (isOpen) {
    setRemoveDOM(false);
  } else {
    setTimeout(() => setRemoveDOM(true), animationDuration + 100);
  }
}, [isOpen]);
```

#### 使用範例

```jsx
const [isOpen, setIsOpen] = useState(false);
const [placement, setPlacement] = useState('left');

<Button onClick={() => { setIsOpen(true); setPlacement('left'); }}>Left</Button>
<Button onClick={() => { setIsOpen(true); setPlacement('right'); }}>Right</Button>

<Drawer
  isOpen={isOpen}
  placement={placement}
  onClose={() => setIsOpen(false)}
>
  <div style={{ width: 300 }}>Drawer 內容</div>
</Drawer>
```

---

### Dropdown

**路徑**：`src/components/Dropdown/index.jsx`

**說明**：下拉選單元件，透過滑鼠事件觸發選單彈出，支援 12 個方位。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `isOpen` | `bool` | `false` | 是否顯示選單 |
| `placement` | 見 Tooltip | `'bottom'` | 出現位置 |
| `overlay` | `string/element` | **必填** | 選單內容 |
| `children` | `string/element` | **必填** | 觸發元件 |
| `onClick` | `func` | `() => {}` | 觸發元件點擊事件 |
| `onClose` | `func` | `() => {}` | 選單關閉事件 |

#### 點擊外部關閉

```jsx
// 透過 data-dropdown-id 屬性判斷是否點擊在 Dropdown 內部
const handleOnClick = useCallback((event) => {
  const dropdownId = findAttributeInEvent(event, 'data-dropdown-id');
  if (!dropdownId) onClose();
}, [onClose]);

document.addEventListener('click', handleOnClick);
```

#### 使用範例

```jsx
const [isOpen, setIsOpen] = useState(false);

<Dropdown
  isOpen={isOpen}
  onClick={() => setIsOpen(true)}
  onClose={() => setIsOpen(false)}
  placement="bottom-left"
  overlay={
    <div style={{ padding: 8 }}>
      <div>選項一</div>
      <div>選項二</div>
    </div>
  }
>
  <Button variant="outlined">Dropdown</Button>
</Dropdown>
```

---

### Pagination

**路徑**：`src/components/Pagination/`

**說明**：分頁元件，當一次要載入過多資料時，分批載入並在不同頁面間切換。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `page` | `number` | `1` | 當前頁數 |
| `pageSize` | `number` | `20` | 每頁資料筆數 |
| `total` | `number` | **必填** | 資料總筆數 |
| `withEllipsis` | `bool` | `false` | 頁碼過多時省略 |
| `themeColor` | `string` | `'primary'` | 主題色 |
| `onChange` | `func` | `() => {}` | 頁碼改變 callback |

#### 使用範例

```jsx
const [page, setPage] = useState(1);

// 基本
<Pagination page={page} total={100} onChange={setPage} />

// 帶省略
<Pagination page={page} pageSize={8} total={100} withEllipsis onChange={setPage} />

// 搭配資料載入
const pageSize = 20;
const handleOnChange = (current) => {
  const max = current * pageSize;
  const min = max - pageSize + 1;
  setDataSource(fakeData.filter((_, i) => i + 1 >= min && i + 1 <= max));
};

<Pagination page={page} pageSize={pageSize} total={fakeData.length} onChange={(c) => { handleOnChange(c); setPage(c); }} />
```

---

### Tabs

**路徑**：`src/components/Tabs/`

**說明**：選項卡切換元件，在同一層級的內容組別間導航切換，由 `Tab`（標籤）+ `TabGroup`（容器 + 底部指示線）組成。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `options` | `array` | `[]` | 選項（含 `value`、`label`） |
| `value` | `string` | `''` | 目前選中的 Tab |
| `themeColor` | `string` | `'primary'` | 主題色 |
| `onChange` | `func` | `() => {}` | 切換時 callback |

#### 核心邏輯（TabGroup）

- 底部指示線（Indicator）透過 `offsetLeft`、`offsetWidth` 動態計算位置與寬度。
- `useEffect` 監聽 `resize` 事件更新 Tab 屬性。
- `transition: all 200ms` 讓指示線滑動。

```jsx
const Indicator = styled.div`
  position: absolute;
  bottom: 0px;
  left: ${props => props.$left}px;
  width: ${props => props.$width}px;
  height: 2px;
  background: ${props => props.$color};
  transition: all 200ms cubic-bezier(0.4, 0, 0.2, 1) 0ms;
`;
```

#### 使用範例

```jsx
const tabOptions = [
  { value: 'item-one', label: 'ITEM ONE' },
  { value: 'item-two', label: 'ITEM TWO' },
  { value: 'item-three', label: 'ITEM THREE' },
];

const [selectedValue, setSelectedValue] = useState(tabOptions[0].value);

const StyledTabs = styled(Tabs)`
  border-bottom: 1px solid #EEE;
`;

<StyledTabs
  value={selectedValue}
  options={tabOptions}
  onChange={(value) => setSelectedValue(value)}
/>
<div style={{ padding: '20px 0' }}>
  {`TabPanel of #${selectedValue}`}
</div>

// 置中對齊
const CenteredTabs = styled(Tabs)`
  border-bottom: 1px solid #EEE;
  .tab__tab-group { justify-content: center; }
`;

// 圖示 Tab
const iconOptions = [
  { value: 'phone',    label: <PhoneIcon /> },
  { value: 'favorite', label: <FavoriteIcon /> },
];
```

---

## 反饋元件

### FormControl

**路徑**：`src/components/FormControl/index.jsx`

**說明**：表單控制器，將 label、required、error 等共用邏輯獨立管理，使被控制的子元件樣式保持一致。適用於 `TextField`、`Switch`、`Checkbox`、`Radio` 等。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `label` | `string/element` | `''` | 標題 |
| `isRequired` | `bool` | `false` | 是否為必填（顯示 `*`） |
| `isError` | `bool` | `false` | 錯誤樣式 |
| `errorMessage` | `string` | `null` | 錯誤訊息 |
| `maxLength` | `number` | `null` | 限制最大輸入長度（顯示計數） |
| `placement` | 見下表 | `'top-left'` | 標題位置 |
| `onChange` | `func` | `() => {}` | 值改變 callback |
| `children` | `element` | `null` | 要管理的 form 元件 |

#### placement 可選值

```
top-left   top   top-right
left              right
bottom-left  bottom  bottom-right
```

#### 核心邏輯

- `Switch` 元件特殊處理（不傳入 `isError`、`value`、`onChange`）。
- 使用 `React.cloneElement` 注入 `isError`、`value`、`onChange` 給子元件。
- `maxLength` 模式下追蹤 `childrenValue` 長度並顯示計數。

#### 使用範例

```jsx
// 基本 Label
<FormControl label="請輸入資料">
  <TextField placeholder="請輸入" />
</FormControl>

// 必填
<FormControl label="請輸入資料" isRequired>
  <TextField placeholder="請輸入" />
</FormControl>

// 限制長度
<FormControl label="請輸入資料" maxLength={12}>
  <TextField placeholder="請輸入" />
</FormControl>

// 錯誤訊息
<FormControl label="請輸入資料" isError errorMessage="請檢查輸入是否錯誤">
  <TextField placeholder="請輸入" />
</FormControl>

// 標題在左邊
<FormControl placement="left" label="標題">
  <TextField placeholder="請輸入" />
</FormControl>

// 搭配 Switch
<FormControl placement="left" label="是否正在載入中">
  <Switch isChecked={isChecked} onChange={() => setIsChecked(prev => !prev)} />
</FormControl>
```

---

### Modal

**路徑**：`src/components/Modal/index.jsx`

**說明**：彈出框基礎元件，為 `Dialog`、`Drawer`、`Dropdown` 等元件提供基礎建設。使用時機：需要用戶處理額外事務，但不希望跳轉頁面打斷流程。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `isOpen` | `bool` | `false` | 是否顯示 |
| `hasMask` | `bool` | `true` | 是否顯示遮罩 |
| `onClose` | `func` | `() => {}` | 關閉事件 |
| `animationDuration` | `number` | `200` | 動畫時間（ms） |
| `children` | `string/element` | **必填** | 彈出框內容 |

#### 使用範例

```jsx
const [isOpen, setIsOpen] = useState(false);

<Button onClick={() => setIsOpen(true)}>Open Modal</Button>
<Modal isOpen={isOpen} onClose={() => setIsOpen(false)}>
  <div style={{ background: '#FFF', padding: 20 }}>Modal 內容</div>
</Modal>
```

---

### Dialog

**路徑**：`src/components/Dialog/`

**說明**：基於 `Modal` 的對話框元件，提供標準的 header（標題 + 關閉按鈕）、content、footer（取消 + 確認按鈕）結構。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `isOpen` | `bool` | `false` | 是否顯示 |
| `title` | `string/element` | `null` | 標題 |
| `onClose` | `func` | `() => {}` | 關閉事件 |
| `onSubmit` | `func` | `() => {}` | 確認事件 |
| `children` | `string/element` | **必填** | 對話框主體內容 |

#### 使用範例

```jsx
const [isOpen, setIsOpen] = useState(false);

<Button onClick={() => setIsOpen(true)}>Open Dialog</Button>
<Dialog
  isOpen={isOpen}
  title={<div style={{ fontWeight: 500 }}>確認刪除</div>}
  onClose={() => setIsOpen(false)}
  onSubmit={() => { /* 執行刪除 */ setIsOpen(false); }}
>
  <div>確定要刪除這筆資料嗎？</div>
</Dialog>
```

---

### ProgressBar

**路徑**：`src/components/ProgressBar/index.jsx`

**說明**：進度條元件，顯示百分比進度，可緩解用戶等待焦慮或提供任務完成成就感。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `value` | `number` | `0` | 進度（0 ~ 100，超出自動截斷） |
| `themeColor` | `string` | `'primary'` | 主題色 |
| `showInfo` | `bool` | `true` | 是否顯示百分比數值 |
| `isStatusActive` | `bool` | `false` | 是否顯示等待光暈動畫 |

#### 核心 CSS

```jsx
// 等待動畫（光暈從左滑到右）
const slide = keyframes`
  from { left: -150%; }
  to   { left: 100%;  }
`;

const activeAnimation = css`
  position: relative;
  overflow: hidden;
  &:before {
    content: '';
    position: absolute;
    height: 100%;
    width: 80px;
    background: linear-gradient(to right, transparent 0%, #FFFFFF99 50%, transparent 100%);
    animation: ${slide} 1s cubic-bezier(0.4, 0.0, 0.2, 1) infinite;
  }
`;
```

#### 使用範例

```jsx
// 基本
<ProgressBar value={75} />

// 隱藏數值
<ProgressBar value={50} showInfo={false} />

// 等待動畫
<ProgressBar value={30} isStatusActive />

// 漸層（透過 styled 覆寫）
const GradientProgressBar = styled(ProgressBar)`
  .progress-bar__track {
    background: linear-gradient(45deg, #FF8E53 30%, #FE6B8B 90%);
  }
`;
<GradientProgressBar value={75} />
```

---

### ProgressCircle

**路徑**：`src/components/ProgressCircle/index.jsx`

**說明**：圓形進度元件，在空間不足的排版中比 ProgressBar 更省空間。使用 SVG `stroke-dasharray` 實現進度弧長。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `value` | `number` | `0` | 進度（0 ~ 100） |
| `themeColor` | `string` | `'primary'` | 主題色 |
| `isClockwise` | `bool` | `true` | 是否順時針（false 為逆時針） |
| `strokeColor` | `object` | `undefined` | 漸層色（`{ '0%': '#108ee9', '100%': '#87d068' }`） |

#### SVG 進度原理

```jsx
// 弧長計算
const perimeter = radius * 2 * Math.PI;      // 圓周長
const argLength = perimeter * (value / 100); // 目前弧長

// stroke-dasharray 控制顯示的弧長
stroke-dasharray: ${argLength} ${INFINITE};  // INFINITE = 999999
```

#### 使用範例

```jsx
<ProgressCircle value={75} />

// 逆時針
<ProgressCircle value={50} isClockwise={false} />

// 漸層
<ProgressCircle
  value={75}
  strokeColor={{ '0%': '#108ee9', '100%': '#87d068' }}
/>

// 自訂大小
const BigCircle = styled(ProgressCircle)`width: 200px; height: 200px;`;
<BigCircle value={87} />
```

---

### Spin

**路徑**：`src/components/Spin/index.jsx`

**說明**：載入狀態元件，當頁面正在處理非同步行為時顯示載入指示器，可包裹子元件作為容器（顯示遮罩 + 居中指示器）。

| Prop | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `indicator` | `element` | `<CircularProgress />` | 自訂載入符號 |
| `isLoading` | `bool` | `false` | 是否載入中 |
| `children` | `string/element` | `''` | 被遮罩的內容 |

#### 核心邏輯

- 無 `children`：直接顯示 indicator。
- 有 `children`：`isLoading` 時在子元件上覆蓋半透明遮罩，並將 indicator 居中。
- 使用 `useRef` + `useEffect` 量測 indicator 尺寸以精確置中。

#### 使用範例

```jsx
// 獨立 spinner
<Spin />

// 自訂 indicator
const RotateContainer = styled.div`
  width: 40px; height: 40px;
  animation: ${rotateAnimation} 1000ms ease-in-out infinite;
`;
<Spin indicator={<RotateContainer><FaSpinner /></RotateContainer>} />

// 包裹容器（isLoading 時顯示遮罩）
const [isLoading, setIsLoading] = useState(false);
<Spin isLoading={isLoading}>
  <div style={{ padding: 20 }}>
    <h1>Lorem Ipsum</h1>
    <p>頁面內容...</p>
  </div>
</Spin>
```

---

### Toast

**路徑**：`src/components/Toast/index.jsx`

**說明**：輕量級訊息提示元件，提供用戶操作反饋（成功、資訊、警告、錯誤），預設頂部置中顯示並自動消失，不打斷用戶操作。

#### 使用方式（命令式 API）

```jsx
import { message } from 'components/Toast';

// 四種類型
message.success({ content: '新增成功', duration: 3000 });
message.info({ content: 'Some information' });
message.warn({ content: '伺服器出了一點問題' });
message.error({ content: '刪除失敗' });
```

| 參數 | 型別 | 預設值 | 說明 |
|---|---|---|---|
| `content` | `string/element` | **必填** | 提示訊息內容 |
| `duration` | `number` | `3000` | 顯示時間（ms） |

#### 實作原理

```jsx
// 動態建立 DOM 並使用 ReactDOM.render 渲染 Toast
const getContainer = () => {
  // 確保 #toast-root 存在
  // 在 toast-root 內建立容器 div，置中對齊
  return newDiv; // 每個 Toast 各自的容器
};

export const message = {
  success: (props) => render(<Toast {...props} type="success" />, getContainer()),
  // ...
};
```

#### 使用範例

```jsx
// 在 onClick handler 中呼叫
<Button onClick={() => message.success({ content: '儲存成功！' })}>儲存</Button>
<Button onClick={() => message.error({ content: '發生錯誤，請重試', duration: 5000 })}>刪除</Button>
```

---

## 小結與重點整理

### 元件設計模式

| 模式 | 元件範例 | 說明 |
|---|---|---|
| **組合模式**（Compound Components） | `Accordion`、`Card/Meta`、`Breadcrumb/Breadcrumbs` | 父子元件協作，透過 props 傳遞狀態 |
| **薄包裝**（Thin Wrapper） | `Checkbox`、`Radio` | 共用底層 `Option` 元件，只替換圖示 |
| **透傳 props**（Props Forwarding） | `Skeleton`、`ColorBlock` | 將 `...props` 透傳到 styled 元件 |
| **命令式 API** | `Toast.message` | 使用 `ReactDOM.render` 動態建立元件 |
| **Portal 模式** | `Modal`、`Drawer`、`Tooltip`、`Dropdown` | 渲染到 body 最上層，避免 z-index 問題 |

### 動畫技術

| 技術 | 用途範例 |
|---|---|
| `keyframes` + `animation` | Drawer 滑入滑出、Toast 飛入飛出、ProgressBar 光暈 |
| `transition` | Button hover、Switch 滑塊、Tabs 指示線滑動、Accordion 展開 |
| `max-height` 動畫 | Accordion panel 展開收合 |
| `stroke-dasharray` | ProgressCircle SVG 弧長進度 |
| CSS `transform` | Carousel 圖片切換、Badge 位置 |

### Styled-Components 技巧

```jsx
// 1. 動態 props（以 $ 前綴避免 DOM 警告）
const StyledButton = styled.button`
  background: ${props => props.$btnColor};
`;

// 2. css helper 封裝可復用樣式片段
const containedStyle = css`background: ${props => props.$color};`;

// 3. 繼承並覆寫樣式
const CustomTable = styled(Table)`
  td, th { border: none; padding: 16px; }
`;

// 4. 存取 theme
const ErrorMessage = styled.div`
  color: ${props => props.theme.color.error};
`;
```

### 常用 React 技巧

```jsx
// React.cloneElement — 注入額外 props 給子元件
React.cloneElement(children, { onClick: handleOnClickUpload })

// React.Children.map — 遍歷 children
React.Children.map(children, (child, index) => ...)

// React.Children.count
const count = React.Children.count(children);

// useRef + getBoundingClientRect — 取得元素位置
const rect = ref.current.getBoundingClientRect();
setPosition({ top: rect.top, left: rect.left });

// useLayoutEffect — DOM 量測（在 render 後同步執行）
useLayoutEffect(() => {
  setLabelWidth(labelRef.current.clientWidth);
}, [labelRef.current.clientWidth]);
```

---

*最後更新：整合自 src/components、src/stories、src/hooks、src/theme、src/utils*
