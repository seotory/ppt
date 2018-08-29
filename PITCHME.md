code-line-numbers : true

# React api 연동해보기



---

# code test

```
import React from 'react';

import PriceCounter from '../common/PriceCounter';

import './SelectedOption.css';

const SelectedOption = ({
  title = "선택항목",
  items,
  selectItemRemove,
  selectItemCntPlus,
  selectItemCntMinus
}) => (
  (items.length > 0)
  &&
  <div className="choice-items">
    {title && <strong>{title}</strong>}
    <ul>
      {items.map((item, idx) =>
        <li className="choice-item clearfix" key={item.itemOption}>
          <div className="pull-left">
            <div className="btn-group" role="group" aria-label="...">
              <button type="button" className="close" aria-label="Close" onClick={e => selectItemRemove(idx)}>
                <span aria-hidden="true">&times;</span>
              </button>
            </div>
            {' '}
            <span>{item.itemOption}</span>
          </div>
          <PriceCounter 
            plus={e => selectItemCntPlus(idx)}
            minus={e => selectItemCntMinus(idx)}
            count={item.itemCount}
            totalPrice={item.itemPrice * item.itemCount}
          />
        </li>
      )}
    </ul>
  </div>
);

export default SelectedOption;
```

---

### Flux Design

- Dispatcher: Manages Data Flow
- Stores: Handle State & Logic
- Views: Render Data via React

---

![Flux Explained](https://facebook.github.io/flux/img/flux-simple-f8-diagram-explained-1300w.png)