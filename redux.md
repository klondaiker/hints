Чистый React
```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import _ from 'lodash';

class Tasks extends React.Component {
  constructor() {
    this.state = { text: '', tasks: [] }
  }

  onChangeText = (e) => {
    e.preventDefault();
    
    this.setState({ text: e.target.value });
  }

  onSubmitForm = (e) => {
    e.preventDefault();

    const { text, tasks } = this.state;

    const newTask = { id: _.uniqueId(), text: text }
    
    this.setState({tasks : [ newTask, ...tasks ], text: '' })
  }

  render() {
    const { text, tasks } = this.state;
  
    return (
      <div>
        <form action="" className="form-inline" onSubmit={onSubmitForm}>
          <div className="form-group mx-sm-3">
            <input type="text" required value={text} onChange={this.onChangeText} />
          </div>
          <input type="submit" className="btn btn-primary btn-sm" value="Add" />
        </form>
        <div className="mt-3">
          <ul className="list-group">
            {tasks.map(({ id, text }) => (
              <li key={id} className="list-group-item d-flex">
                <span className="mr-auto">{text}</span>
              </li>
            ))}
          </ul>
        </div>
       </div>
    );
  }
}

ReactDOM.render(
  <Tasks />,
  document.getElementById('container');,
);
```

Чистый Redux
```jsx
import ReactDOM from 'react-dom';
import React from 'react';
import { createStore } from 'redux';
import _ from 'lodash';

const reducer = (state = { text: '', tasks: [] }, action) => {
  switch(action.type) {
    case 'TEXT_CHANGE':
      return { text: action.payload.text, tasks: state.tasks };
    case 'TASK_ADD':
      return { text: '', tasks: [newTask, ...tasks] };
    default:
      return state;
  }
}

const addTaskAction = (task) => ({
  type: 'TASK_ADD',
  payload: { task },
});

const changeTextAction = (text) => ({
  type: 'TEXT_CHANGE',
  payload: { text },
});

const Tasks = ({dispatch, text, tasks}) => {
  const onChangeText = (e) => {
    e.preventDefault();

    dispatch(changeTextAction(e.target.value));
  }

  const onSubmitForm = (e) => {
    e.preventDefault();

    const newTask = { id: _.uniqueId(), text: text };
    dispatch(addTaskAction(newTask));
  }

  return (
    <div>
      <form action="" className="form-inline" onSubmit={onSubmitForm}>
        <div className="form-group mx-sm-3">
          <input type="text" required value={text} onChange={this.onChangeText} />
        </div>
        <input type="submit" className="btn btn-primary btn-sm" value="Add" />
      </form>
      <div className="mt-3">
        <ul className="list-group">
          {tasks.map(({ id, text }) => (
            <li key={id} className="list-group-item d-flex">
              <span className="mr-auto">{text}</span>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}

const store = createStore(reducer);

store.subscribe(() => {
  const { text, tasks } = store.getState();
  ReactDOM.render(
    <Tasks dispatch={store.dispatch} text={text} tasks={tasks} />,
    containerElement,
  );
});

ReactDOM.render(
  <Tasks dispatch={store.dispatch} text={''} tasks={[]} />,
  document.getElementById('container');,
);
```

Добавил combineReducers, Provider, connect
```jsx
import ReactDOM from 'react-dom';
import React from 'react';
import { createStore } from 'redux';
import { connect, Provider } from 'react-redux';
import _ from 'lodash';

const textReducer = (state = '', action) => {
  switch(action.type) {
    case 'TEXT_CHANGE':
      return action.payload.text;
    case 'TASK_ADD':
      return '';
    default:
      return state;
  }
}

const tasksReducer = (state = [], action) => {
  switch(action.type) {
    case 'TASK_ADD':
      return [action.payload.task, ...state];
    default:
      return state;
  }
}

const reducers = combineReducers({text: textReducer, tasks: tasksReducer});

const addTaskAction = (task) => ({
  type: 'TASK_ADD',
  payload: { task },
});

const changeTextAction = (text) => ({
  type: 'TEXT_CHANGE',
  payload: { text },
});

const mapStateToProps = (state) => {
  const { tasks, text } = state;
  const props = {tasks, text };
  
  return props;
};

const Tasks = connect(mapStateToProps)({ dispatch, text, tasks }) => {
  const onChangeText = (e) => {
    e.preventDefault();

    dispatch(changeTextAction(e.target.value));
  }

  const onSubmitForm = (e) => {
    e.preventDefault();
    const newTask = { id: _.uniqueId(), text: text };
    dispatch(addTaskAction(newTask));
  }

  return (
    <div>
      <form action="" className="form-inline" onSubmit={onSubmitForm}>
        <div className="form-group mx-sm-3">
          <input type="text" required value={text} onChange={this.onChangeText} />
        </div>
        <input type="submit" className="btn btn-primary btn-sm" value="Add" />
      </form>
      <div className="mt-3">
        <ul className="list-group">
          {tasks.map(({ id, text }) => (
            <li key={id} className="list-group-item d-flex">
              <span className="mr-auto">{text}</span>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}

const store = createStore(reducers);

render(
  <Provider store={store}>
    <Tasks />
  </Provider>,
  document.getElementById('container'),
);
```


Добавил reduxActions и actionCreators, 
```jsx
import ReactDOM from 'react-dom';
import React from 'react';
import { createStore } from 'redux';
import { connect, Provider } from 'react-redux';
import { createAction, handleActions } from 'redux-actions';
import _ from 'lodash';

const addTaskAction = createAction('USER_ADD');
const changeTextAction = createAction('TEXT_CHANGE');

const textHandlers = {
  [changeTextAction]: ((state, { payload: { text } }) => text), 
  [addTaskAction]: (() => ''), 
};

const textReducer = handleActions(textHandlers, '');

const tasksHandlers = {
  [addTaskAction]: ((state, { payload: { task } }) => [task, ...state]), 
};

const tasksReducer = handleActions(tasksHandlers, '');

const reducers = combineReducers({text: textReducer, tasks: tasksReducer});

const mapStateToProps = (state) => {
  const { tasks, text } = state;
  const props = {tasks, text };
  
  return props;
};

const actionCreators = {
  addTask: addTaskAction,
  changeText: changeTextAction,
};

const Tasks = connect(mapStateToProps, actionCreators)({ addTask, changeText, text, tasks }) => {
  const onChangeText = (e) => {
    e.preventDefault();

    changeText({ text: e.target.value }));
  }

  const onSubmitForm = (e) => {
    e.preventDefault();
    const newTask = { id: _.uniqueId(), text: text };
    addTask({ task: newTask });addTask
  }

  return (
    <div>
      <form action="" className="form-inline" onSubmit={onSubmitForm}>
        <div className="form-group mx-sm-3">
          <input type="text" required value={text} onChange={this.onChangeText} />
        </div>
        <input type="submit" className="btn btn-primary btn-sm" value="Add" />
      </form>
      <div className="mt-3">
        <ul className="list-group">
          {tasks.map(({ id, text }) => (
            <li key={id} className="list-group-item d-flex">
              <span className="mr-auto">{text}</span>
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}

const store = createStore(reducers);

render(
  <Provider store={store}>
    <Tasks />
  </Provider>,
  document.getElementById('container'),
);
```
