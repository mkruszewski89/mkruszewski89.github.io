---
layout: post
title:      "React Redux with Rails API"
date:       2018-06-20 23:20:09 +0000
permalink:  react_redux_with_rails_api
---


One of the greatest advantages of web development is that there are as many ways to accomplish a task as there are people to accomplish it. My immersive introduction to web development has culminated in developing an app with a React client that uses Redux middleware to handle state and Rails as a backend API. Every feature of React, Redux, and Rails merits its own exploratory blog post, and the best way to learn is to just start coding and take advantage of the phenomenal online community to accomplish tasks as they arise. This blog post will focus on getting React, Redux, and Rails talking to each other so you can get learning figuring out your idiosyncratic tasks!

I assume knowledge of rails - the following code will create a resource to interact with from our client:
console
```
rails new app_api --api
cd app_api
rails g resource model my_attribute:string
```
app_api/app/models/model.rb
```
class Model < ApplicationRecord
  def self.build_json
    json = {}
    all.each{|model|
      json[model.id] = model.build_json}
    json
  end

  def build_json
    {my_attribute: my_attribute}
  end
end
```
app_api/app/controllers/models_controller.rb
```
class ModelsController < ApplicationController
  
  def index
    render json: Model.build_json
  end

  def create
    Model.create(model_params)
  end

  private

  def model_params
    params.require(:model).permit(:my_attribute)
  end

end
```

handle CORS for purposes of this demo:
app_api/Gemfile (add)
```
gem 'rack-cors'
```
app_api/config/application (add)
```
config.middleware.insert_before 0, Rack::Cors do
      allow do
        origins 'http://localhost:3000'
        resource '*', :headers => :any, :methods => [:get, :post, :put, :delete, :options]
      end
    end
```

Now we can boot Rails and see our (temporarily) empty collection of model objects:
console:
```
rails s -p 3001
```
browser:
```
http://localhost:3001/models
```

With the API up and running let's create the react client:
console:
```
npx create-react-app app-client
cd app-client
npm install react-redux
npm install redux
npm install redux-thunk
```
app-client/src/index.js
```
import React from 'react'
import ReactDOM from 'react-dom'
import {Provider} from 'react-redux'
import {createStore, applyMiddleware} from 'redux'
import modelReducer from './modelReducer'
import thunk from 'redux-thunk'
import App from './App'

const store = createStore(modelReducer, applyMiddleware(thunk))
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root'))
```
app-client/src/modelReducer.js
```
export default function modelReducer(
  state = {
    my_attribute: '',
  }, action) {
  switch ( action.type ) {

    case 'MODIFY_MODEL_MY_ATTRIBUTE':
      return {...state,
        my_attribute: action.payload}

    default:
      return state}}
```
app-client/src/modelReducerActions.js
```
export function modifyModelMyAttribute(value) {
  return (dispatch) => {
    dispatch({
      type: 'MODIFY_MODEL_MY_ATTRIBUTE',
      payload: value})}}
```
app-client/src/apiActions.js
```
export function writeModel(method, my_attribute) {
  let preparedModel = {model: {my_attribute: my_attribute}}
  return fetch('http://localhost:3001/models', {
    method: method,
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json',},
    body: JSON.stringify(preparedModel)})}
```
app-client/src/App.js
```
import React, { Component } from 'react'
import {connect} from 'react-redux'
import {bindActionCreators} from 'redux'
import {modifyModelMyAttribute} from './modelReducerActions'
import {writeModel} from './apiActions'

const mapStateToProps = (state) => {
  return {
    my_attribute: state.my_attribute}}

const mapDispatchToProps = (dispatch) => {
  return bindActionCreators({
    modifyModelMyAttribute: modifyModelMyAttribute,
  }, dispatch)}

class App extends Component {
  render() {
    return (
      <form onSubmit={(event) => {
        event.preventDefault()
        writeModel('post', this.props.my_attribute)}}>
      <input
        type="text"
        placeholder="input my_attribute"
        value={this.props.my_attribute}
        onChange={(event) => {this.props.modifyModelMyAttribute(event.target.value)}} />
      <input type="submit" />
      </form>)}}

export default connect(mapStateToProps, mapDispatchToProps)(App)
```

Now when you run npm start you will see a very basic form for creating a model instance with the sole attribute we defined. Hit submit and refresh http://localhost:3001/models to see the new record hit your API!

This very basic set up includes all of the nuts-and-bolts for a React client using Redux middleware and a Rails backend. Set up this way, you are well positioned to break things and learn through fixing. Happy hacking!
