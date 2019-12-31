# Save Redux State On Refresh

- [Save Redux State On Refresh](#save-redux-state-on-refresh)
  - [Project Structure](#project-structure)
  - [Guide](#guide)
    - [Listen on state change](#listen-on-state-change)
    - [Restore state on refresh](#restore-state-on-refresh)


This guide will describle how to save redux state to `localStorage` and restore it on refresh.

> Example Project https://github.com/hieu-pv/vicoders-cheatsheet-angular-example/tree/Save_Redux_State_On_Refresh

## Project Structure



```
.
├── app
│   ├── app-injector.ts
│   ├── app.component.html
│   ├── app.component.scss
│   ├── app.component.spec.ts
│   ├── app.component.ts
│   ├── app.module.ts
│   ├── components
│   │   ├── base.component.ts
│   │   ├── components.module.ts
│   │   ├── main.component.html
│   │   ├── main.component.scss
│   │   ├── main.component.ts
│   │   └── master.component.ts
│   └── store
│       ├── action.ts
│       ├── reducers.ts
│       ├── sagas.ts
│       └── store.module.ts
├── assets
├── environments
│   ├── environment.prod.ts
│   └── environment.ts
├── favicon.ico
├── index.html
├── main.ts
├── polyfills.ts
├── styles.scss
└── test.ts
```

## Guide

### Listen on state change

Insert the code bellow to an controller `{component}.component.ts`

For example

`app.component.ts`

```javascript

import { Component, OnInit } from '@angular/core';
import { Store } from './store/store.module';
import initSubscriber from 'redux-subscriber';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent implements OnInit {
  public title = 'app';
  public store;
  unsubscribe: any;
  constructor(store: Store) {
    this.store = store.getInstance();
  }

  ngOnInit(): void {
    const saveState = state => {
      try {
        const serializedState = JSON.stringify(state);
        localStorage.setItem('State', serializedState);
      } catch {}
    };
    const subscribe = initSubscriber(this.store);

    // Listen on change of state
    const StateToListen = 'App';
    this.unsubscribe = subscribe(StateToListen, state => {
      saveState(state.App);
    });
  }

  change() {
    this.store.dispatch({ type: 'CHANGE', data: 'new value' });
  }

  reset() {
    localStorage.removeItem('State');
    console.log('localStorage is cleared');
  }
}


```

### Restore state on refresh

In reducer file we just need to restore data that is saved to localStorage before

`reducers.ts`

```
import * as _ from 'lodash';
import { combineReducers } from 'redux';

let defaultState = { initValue: 'NF' };

if (!_.isNil(localStorage.getItem('State'))) {
  const loadState = state => {
    try {
      const serializedState = localStorage.getItem(state);
      if (serializedState === null) {
        return undefined;
      }
      return JSON.parse(serializedState);
    } catch (err) {
      return undefined;
    }
  };
  defaultState = _.assign(loadState('State'), { is_restored: true });
}

const App = (state = defaultState, action) => {
  switch (action.type) {
    case 'CHANGE':
      return _.assign({}, state, { initValue: action.data });
    default:
      return state;
  }
};

export default combineReducers({ App });

```