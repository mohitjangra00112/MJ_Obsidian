## Redux -> Independent Library

###### Store ->  Single source of truth , where we store data or we can say (Global Storage)

### Reducers 
### Methods  -->  useSelector  ,   useDispatch 
( We only need to study these two methods , *useSelector* is used to select the data and the *useDispatch* is used to send data   )

## Install

npm install @reduxjs/toolkit
npm install react-redux

#### Create a Store  
Create app folder inside the src and then create a file store.js

### Create another file Slice
Redux_Todo\src\features\todo\todoSlice.js

```js
import { createSlice , nanoid } from "@reduxjs/toolkit";
// nanoid provides the unique id 
```

#### Note -> every slice has a initial state.

#### Reducers contains properties and functions 