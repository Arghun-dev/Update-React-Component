# Update-React-Component

One of the most important topics in react which annoyed me for 3 months. finally I learned.

Whenever you want to call an AJAX call based on state change, for example imagine you have 2 dropdowns and you want to call and Ajax call based on every change in states of at least one of these dropdowns:

**This is very important to remember that you should `never` `setState` inside `componentDidUpdate`**
**To update component based on state changes define a function for example `callFetchRouteTracking_List` and pass it inside event listener of that dropdown or button or ...**

just like below:

```js
<Select
    size="middle"
    showSearch
    style={{ width: '95.5%', marginRight: '9.5%' }}
    placeholder="انتخاب کاربر"
    optionFilterProp="children"
    onChange={val => {
        this.setState({ selectedUser: val })
        setTimeout(() => {
            this.callFetchRouteTracking_List()
        }, 1)
    }}
    filterOption={(input, option) =>
        option.children.toLowerCase().indexOf(input.toLowerCase()) >= 0
    }
>
    {this.state.users}
</Select>
```

As you can see I called `callFetchRouteTracking_List` inside the event listener of `Select` component. And I called this function inside `setTimeout` to be sure that the selectedUser setState completed.

And We will do this exactly to the other inputs with the same `callFetchRouteTracking_List` function. 

**finally, we will call the Ajax call inside this method which we have defined (`callFetchRouteTracking_List`)**

Full Example: 

```js
import React, { Component } from 'react';
import { Row, Col } from 'reactstrap';
import Mapir from "mapir-react-component";

// Antd Components
import { Select } from 'antd';
import "react-modern-calendar-datepicker/lib/DatePicker.css";
import DatePicker from "react-modern-calendar-datepicker";

// Style
import './VisitorTracking.scss';

// Redux
import { connect } from 'react-redux';
import { RouteTracking_List } from '../../../actions/dashboard/marketingAndSale/visitorTracking/RouteTracking_List';
import { getUsersList } from '../../../actions/dashboard/systemmanagement/groups/Users';

const { Option } = Select;

const Map = Mapir.setToken({
    transformRequest: (url) => {
        return {
            url: url,
            headers: {
                'x-api-key': 'eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImp0aSI6IjZjZTc2MzY1MGZmYzhlNDkwZjE5ZDQxMzUyN2Y3N2UwYThhY2I4ZTY5ZGY2ODc5N2Q4YTgyOWNjOTVlZWYzODgzZjU2OGFkOWJjYTRmNzg5In0.eyJhdWQiOiI5ODczIiwianRpIjoiNmNlNzYzNjUwZmZjOGU0OTBmMTlkNDEzNTI3Zjc3ZTBhOGFjYjhlNjlkZjY4Nzk3ZDhhODI5Y2M5NWVlZjM4ODNmNTY4YWQ5YmNhNGY3ODkiLCJpYXQiOjE1OTM0MDYyMTcsIm5iZiI6MTU5MzQwNjIxNywiZXhwIjoxNTk1OTk4MjE3LCJzdWIiOiIiLCJzY29wZXMiOlsiYmFzaWMiXX0.FruNzI0GKLZdMt7MMIqP2nBppPTeONVLFgfcf9MIohzGp6bL9lrVYoMEYcIdPE3BmTEt9JHNHBaw-jg7FuwLU8itnx7viEXmbRa2MLS3otCvQzHIGAZNG2m8CFREeOaCuz1Yo3Hq5kcgsA9n_RwWnkQszi12u0n-sSmjeSwMCeMk8rVrcBK7ZnOGR4IX_gH_XihCTgx-tKpxzOvrKXlKiDlwoTUQdQBQzMudLL94GwBqR2t4yfTBCabVfg1EHzbwvWkXTKYcV8Wc1rEIQJYc8KM3lrSruWEdg1BUP9Ev4-iS9m-5w0u8xD-ljFLIYWJw6tDX7VqTI7m1TEomlLkcPw', //Mapir api key
                'Mapir-SDK': 'reactjs'
            }
        }
    }
});

class VisitorTracking extends Component {
    constructor(props) {
        super(props);

        this.state = {
            markers: [],
            users: [],
            selectedUser: null,
            selectedDate: { from: null, to: null },
            usersLoaded: false
        }
    }

    // ========== Correct Form of from date ==========
    correctFormOfSelectedDateFrom = () => {
        // ===== Destructuring =====
        // const { year, month, day } = this.state.selectedDate.from
        const correctForm = `${this.state.selectedDate.from?.year}${('0' + this.state.selectedDate.from?.month).slice(-2)}${('0' + this.state.selectedDate.from?.day).slice(-2)}`
        return correctForm
    }

    // ========== Correct Form of to date ==========
    correctFormOfSelectedDateTo = () => {
        // ===== Destructuring =====
        // const { year, month, day } = this.state.selectedDate.to
        const correctForm = `${this.state.selectedDate.to?.year}${('0' + this.state.selectedDate.to?.month).slice(-2)}${('0' + this.state.selectedDate.to?.day).slice(-2)}`
        return correctForm
    }

    componentDidMount() {
        if (this.state.selectedUser && this.state.selectedDate.from && this.state.selectedDate.to) {
            this.props.RouteTracking_List(this.state.selectedUser, this.correctFormOfSelectedDateFrom(), this.correctFormOfSelectedDateTo(), this.props.token);
        }
        this.props.getUsersList(this.props.token);
    }

    // Call RouteTracking_List method again based on input changes
    callFetchRouteTracking_List = () => {
        if (this.state.selectedUser && this.state.selectedDate.from && this.state.selectedDate.to) {
            this.props.RouteTracking_List(this.state.selectedUser, this.correctFormOfSelectedDateFrom(), this.correctFormOfSelectedDateTo(), this.props.token);
        }
    }

    fetchRouteTracking_List = () => {
        if (this.props.RouteTracking_List_Msg.RouteTracking_List?.length > 0) {
            this.props.RouteTracking_List_Msg.RouteTracking_List.map(trackItem =>
                this.state.markers.push(
                    <Mapir.Marker
                        coordinates={[trackItem.lng, trackItem.Lat]}
                        anchor="bottom"
                    >
                    </Mapir.Marker>
                )
            )
        }
    }

    fetchUsers = () => {
        if (this.props.users.users?.resultBody?.length > 0 && this.state.usersLoaded === false) {
            this.setState({
                users: this.props.users.users.resultBody.map(user => (
                    <Option key={user.UserId} value={user.UserId}>{user.UserDescription}</Option>
                )),
                usersLoaded: true
            })

        }
    }

    render() {
        this.fetchRouteTracking_List();
        this.fetchUsers();

        console.log(this.state.markers)
        return (
            <div className='visitorTrackingPage'>
                <Row>
                    <Col md='2' className='visitorTrackingPage__settings-column'>
                        <label style={{ marginTop: '15%', marginRight: '9.5%' }}>انتخاب کاربر</label>
                        <Select
                            size="middle"
                            showSearch
                            style={{ width: '95.5%', marginRight: '9.5%' }}
                            placeholder="انتخاب کاربر"
                            optionFilterProp="children"
                            onChange={val => {
                                this.setState({ selectedUser: val })
                                setTimeout(() => {
                                    this.callFetchRouteTracking_List()
                                }, 1)
                            }}
                            filterOption={(input, option) =>
                                option.children.toLowerCase().indexOf(input.toLowerCase()) >= 0
                            }
                        >
                            {this.state.users}
                        </Select>
                        <label style={{ marginTop: '15%', marginRight: '9.5%' }}>انتخاب بازه زمانی</label>
                        <DatePicker
                            calendarPopperPosition='auto'
                            wrapperClassName='date-picker-container'
                            inputClassName='custom-input'
                            locale="fa"
                            calendarClassName="responsive-calendar"
                            value={this.state.selectedDate}
                            onChange={val => {
                                this.setState({ selectedDate: val })
                                setTimeout(() => {
                                    this.callFetchRouteTracking_List()
                                }, 1)
                            }}
                            shouldHighlightWeekends
                        />
                    </Col>
                    <Col md='10'>
                        <Mapir
                            center={[51.420470, 35.729054]}
                            Map={Map}
                            apiKey={'eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImp0aSI6IjZjZTc2MzY1MGZmYzhlNDkwZjE5ZDQxMzUyN2Y3N2UwYThhY2I4ZTY5ZGY2ODc5N2Q4YTgyOWNjOTVlZWYzODgzZjU2OGFkOWJjYTRmNzg5In0.eyJhdWQiOiI5ODczIiwianRpIjoiNmNlNzYzNjUwZmZjOGU0OTBmMTlkNDEzNTI3Zjc3ZTBhOGFjYjhlNjlkZjY4Nzk3ZDhhODI5Y2M5NWVlZjM4ODNmNTY4YWQ5YmNhNGY3ODkiLCJpYXQiOjE1OTM0MDYyMTcsIm5iZiI6MTU5MzQwNjIxNywiZXhwIjoxNTk1OTk4MjE3LCJzdWIiOiIiLCJzY29wZXMiOlsiYmFzaWMiXX0.FruNzI0GKLZdMt7MMIqP2nBppPTeONVLFgfcf9MIohzGp6bL9lrVYoMEYcIdPE3BmTEt9JHNHBaw-jg7FuwLU8itnx7viEXmbRa2MLS3otCvQzHIGAZNG2m8CFREeOaCuz1Yo3Hq5kcgsA9n_RwWnkQszi12u0n-sSmjeSwMCeMk8rVrcBK7ZnOGR4IX_gH_XihCTgx-tKpxzOvrKXlKiDlwoTUQdQBQzMudLL94GwBqR2t4yfTBCabVfg1EHzbwvWkXTKYcV8Wc1rEIQJYc8KM3lrSruWEdg1BUP9Ev4-iS9m-5w0u8xD-ljFLIYWJw6tDX7VqTI7m1TEomlLkcPw'}
                        >
                            <Mapir.Layer
                                type="symbol"
                                layout={{ "icon-image": "harbor-15" }}>
                            </Mapir.Layer>

                            {this.state.markers}
                        </Mapir>
                    </Col>
                </Row>
            </div>
        )
    }
}

const mapStateToProps = state => {
    return {
        token: state.auth[0].token,
        RouteTracking_List_Msg: state.RouteTracking_List,
        users: state.users
    }
}

export default connect(mapStateToProps, { RouteTracking_List, getUsersList })(VisitorTracking);
```
