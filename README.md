# react-redux-polling
Just an example of polling using React-Redux using ReactiveX

## example

This is an example container that queries the list of Users periodically.

The polling container executes the redux action `pollUsers` every 2 seconds.

To use the `PollingContainer` component you must install https://www.npmjs.com/package/rxjs


### container

```javascript
import React, {Component} from "react";
import {connect} from "react-redux";
import {bindActionCreators} from "redux";
import { pollUsers} from "../../actions/userActions";
import PollingContainer from "../../genericContainers/PollingContainer";
import { ToastContainer, toast } from 'react-toastify';
import 'react-toastify/dist/ReactToastify.css';
import UserList from "../../components/users/UserList";


class UsersContainer extends Component{

    render=()=>{
        return <div>
            <PollingContainer
                render={this.renderPolling}
                pollAction={this.props.pollUsers}
                dueTim={0}
                periodOfScheduler={2000}
            />
            <ToastContainer />
            <div className="container-fluid px-5">
                <UserList userList={this.props.users}/>
            </div>
        </div>

    };


    renderPolling=()=>{
        if(this.props.usersChanged){
            toast.info("Users changed")
        }
    };


}

const mapStateToProps = (state) => {
    return {
        users: state.userslReducer.users,
        usersChanged: state.usersReducer.usersChanged
    }
};

const mapDispatchToProps = (dispatch) => {
    const actions = {
        pollUsers: pollUsers
    };
    return bindActionCreators(actions, dispatch)
};

export default connect(mapStateToProps, mapDispatchToProps)(UsersContainer);
```
 
### action

```javascript

import axios from 'axios';
import {POLL_USERS_SUCCED} from "./types";


export const pollUsers = () =>
    (dispatch, getState) =>
        new Promise((resolve, reject) =>
            axios.get("http://localhost:8080/users").then(response => {
                 return resolve(dispatch(pollSucceed(response.data, true)))); // change data changed to a function that compares with the previous state
            })
            .catch(error => {
                console.log(JSON.stringify(error));
            })
        );
        
const pollSucceed= (users, dataChanged) => (
        {
            type: POLL_USERS_SUCCED,
            data: {users: users, usersChanged: dataChanged}
        }
)
        

```


### reducer


```javascript
import createReducer from './helpers/reducerHelper'
import {POLL_USER_SUCCEED} from "../actions/types";

const initialState = {
    users: [],
    usersDataChanged: false
};

const usersReducer = createReducer(initialState,
    {
        [POLL_CHANNELS](state, action) {
            console.log("Dispatched poll")
            return {
                ...state,
                users:  action.data.channels,
                usersChanged: action.data.usersChanged,

            };
        },

    });


export default usersReducer;
```



