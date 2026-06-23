# 模态对话框

**任务：** “为删除确认添加一个模态对话框。”

## 未启用 Ponytail

```bash
npm install @radix-ui/react-dialog
# 或：npm install react-modal
```

```jsx
import * as Dialog from "@radix-ui/react-dialog";
import { useState } from "react";

export function DeleteModal({ onConfirm, onCancel }) {
  return (
    <Dialog.Root>
      <Dialog.Trigger asChild>
        <button className="btn-danger">Delete</button>
      </Dialog.Trigger>
      <Dialog.Portal>
        <Dialog.Overlay className="dialog-overlay" />
        <Dialog.Content className="dialog-content">
          <Dialog.Title>Confirm deletion</Dialog.Title>
          <Dialog.Description>This action cannot be undone.</Dialog.Description>
          <div className="dialog-actions">
            <Dialog.Close asChild>
              <button onClick={onCancel}>Cancel</button>
            </Dialog.Close>
            <button className="btn-danger" onClick={onConfirm}>Delete</button>
          </div>
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  );
}
```

为了显示一个带两个按钮的框，引入了依赖、portal、overlay、root、trigger 和 content wrapper。

## 启用 Ponytail

```html
<!-- ponytail: 浏览器原生就有，内置焦点锁定和背景遮罩 -->
<dialog id="confirm-delete">
  <p>This action cannot be undone.</p>
  <button id="cancel">Cancel</button>
  <button id="confirm">Delete</button>
</dialog>
```

```js
const dialog = document.getElementById("confirm-delete");
document.getElementById("cancel").onclick = () => dialog.close();
document.getElementById("confirm").onclick = () => { onConfirm(); dialog.close(); };

// 打开：
dialog.showModal();
```

**1 个依赖 + 30 行 → 0 个依赖 + 8 行。** 原生 `<dialog>` 会自动锁定焦点，按 Escape 可关闭，可用 `::backdrop` 渲染背景遮罩，并且默认具备可访问性。2022 年以来所有浏览器都支持。那个库解决的是平台已经解决的问题。
