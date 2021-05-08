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
    
    this.setState({tasks : [ newTask, ...tasks ] })
  }

  render() {
    const { text, tasks } = this.state;
  
    return (
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
      const { text, tasks } = this.state;
      const newTask = { id: _.uniqueId(), text: text };
      return { text, tasks: [newTask, ...tasks] };
    default:
      return state;
  }
}

const store = createStore(reducer);

const addTask = () => ({
  type: 'TASK_ADD',
  payload: {},
});

const changeText = (text) => ({
  type: 'TEXT_CHANGE',
  payload: { text },
});

const Tasks = (dispatch, text, tasks) => {
  const onChangeText = (e) => {
    e.preventDefault();

    dispatch(changeText(e.target.value));
  }

  const onSubmitForm = (e) => {
    e.preventDefault();
  
    dispatch(addTask())
  }

  return (
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
  );
}

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
