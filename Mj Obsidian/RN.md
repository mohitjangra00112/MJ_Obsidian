## Note  
We have to use StyleSheet for css instead of JS Objects because when we render pages the StyleSheet loads only one time but the JS Objects loads every time we render page . This make application show .

#### Alignitems does not  work with wrap , so use aligncontent 

metro.config.js  -> This file helps in fast reload of application 
babel.config.jd  -> This file converts latest code to old code so thar rhe browser can understand it . 

note -> ek component ki state uske ander hi rahegi uske bhar nhi ja sakti.
Note -> We cannot share the data of state with other components but we can share data of props with other components.
Props are arguments of components to send data to that component.
We can apply more than one stylesheet by writing them in array style={ [ s1 , s2  , { margin: 20 } ]  }



