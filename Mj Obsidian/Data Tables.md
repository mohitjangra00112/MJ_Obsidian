### Objects

If you are actively working with the data through [the API](https://datatables.net/manual/api), objects can make obtaining a particular piece of data very easy as you need only use a property name, rather than remembering which array index that data is in (for example `data.name`, rather than `data[0]`).

The down side of using objects is that you need to explicitly tell DataTables which property it should use from the object for each column. This is done using the [`columns.data`](https://datatables.net/reference/option/columns.data) and / or [`columns.render`](https://datatables.net/reference/option/columns.render) options.

```js
[
  {
    "name": "Tiger Nixon",
    "position": "System Architect",
    "salary": "$3,120",
    "start_date": "2011/04/25",
    "office": "Edinburgh",
    "extn": "5421"
  },
  {
    "name": "Garrett Winters",
    "position": "Director",
    "salary": "$5,300",
    "start_date": "2011/07/25",
    "office": "Edinburgh",
    "extn": "8422"
  }
]

```

And the table initialisation:

```js
$('#example').DataTable({
    data: data,
    columns: [
        { data: 'name' },
        { data: 'position' },
        { data: 'salary' },
        { data: 'office' }
    ]
});

```

### Instances

It can be quite useful to have DataTables display information from Javascript object instances, as these instances define abstract methods which can be used to update data.

```js
function Employee(name, position, salary, office) {
    this.name = name;
    this.position = position;
    this.salary = salary;
    this._office = office;

    this.office = function () {
        return this._office;
    };
}

$('#example').DataTable({
    data: [
        new Employee("Tiger Nixon", "System Architect", "$3,120", "Edinburgh"),
        new Employee("Garrett Winters", "Director", "$5,300", "Edinburgh")
    ],
    columns: [
        { data: 'name' },
        { data: 'salary' },
        { data: 'office' },
        { data: 'position' }
    ]
});


```

Note that `office` is a method of the class above, while `name`, `position` and `salary` are properties. DataTables will automatically realise that there is a function, execute it and use the returned value for the cell (note you could also use the syntax `office()` to be explicit that a function is used - see [`columns.data`](https://datatables.net/reference/option/columns.data) for further information).3


## Data sources

- DOM (i.e. the plain HTML of the document)
- Javascript
- Ajax sourced data

#### HTML5

DataTables can also make use of HTML5 `data-*` attributes, which can provide DataTables with additional information for ordering and search data. For example you might have a column with a date formatted such as "21st November 2016". Browsers will struggle to sort that, but you could provide a `data-order` attribute as part of the HTML for the cell which contains a timestamp that can be easily sorted upon. Extending that, search data can be provided using `data-search`. For example:

```html
<td data-search="21st November 2016 21/11/2016" data-order="1479686400">
    21st November 2016
</td>

```

### Javascript

You can instruct DataTables to use data given to it using the [`data`](https://datatables.net/reference/option/data) initialisation option. This data can be in the form of arrays, objects or instances (see above) and can be sourced from anywhere you want! As long as Javascript can access the data, you can send it to DataTables (be it from a custom Ajax call, a WebSocket or just a good old fashioned array of data).

### Ajax

Ajax sourced data is much like Javascript sourced data, but DataTables will make an Ajax call to get the data for you. It can often be very useful to source table data from a specific script, separating the logic for retrieving the data from the display. Ajax sourced data in DataTables is controlled by the [`ajax`](https://datatables.net/reference/option/ajax) option. In its simplest form, you set the property value as a string, pointing at the URL you want to load data from.

[Server-side processing](https://datatables.net/manual/server-side) as discussed above is a special form of Ajax sourced data, whereby the data to be shown for each page in the DataTable is retrieved by an Ajax request only when that page is required for display to the user. 



## other

DataTables will automatically detect the following attributes on HTML cells:

- `data-sort` and `data-order` - for ordering data
- `data-filter` or `data-search` - for search data
```html
<tr>
    <td data-search="Tiger Nixon">T. Nixon</td>
    <td>System Architect</td>
    <td>Edinburgh</td>
    <td>61</td>
    <td data-order="1303682400">Mon 25th Apr 11</td>
    <td data-order="3120">$3,120/m</td>
</tr>

```

# Renderers

There are occasions when the data source for the table's rows does not contain the value that you would want to display in the table. You may wish to transform it to a different representation (such as a time stamp into a human readable format), combine data points (first and last names, perhaps) or perform some computation on the value (maybe calculating margin from turnover and expense values).

## Data rendering

The primary advantage of using a data renderer in DataTables is that you can modify the output data without modifying the original data.

#### Adding formatting

In our DataTable, if we wish to have a column that shows the price, it is relatively common to wish to prefix it with a currency sign. In this case we use a dollar sign (see also the built-in number renderer below which provides advanced formatting options):

```js
{
    data: 'price',
    render: function(data, type, row) {
        return '$' + data;
    }
}

```

#### Joining strings

In our DataTable if we wish to have a single column that shows the full name

```js
{
    data: 'creator',
    render: function(data, type, row) {
        return data.firstName + ' ' + data.lastName;
    }
}

```


## Built-in helpers

DataTables has a number of built in rendering helpers that can be used to easily format data - more can be added using plug-ins (see below):

- `date` - formatting dates (since 1.12)
- `datetime` - formatting date times (since 1.12)
- `number` - for formatting numbers
- `text` - to securely display text from a potentially unsafe source (HTML entities are escaped)
- `time` - formatting times (since 1.12).

## Built in types

You can use the [`DataTable.types()`](https://datatables.net/reference/api/DataTable.types\(\)) method to determine what types are available to tables on the page. By default these are:

- `num` - Plain numbers (e.g. _1_, _8432_).
- `num-fmt` - Formatted numbers (e.g. _$1'000_, _8,000,000_).
- `html-num` - Plain numbers with HTML (e.g. _**10**_).
- `html-num-fmt` - Formatted numbers with HTML (e.g. `_<em>€9.200,00</em>`)
- `date` - Dates in [ISO8601 format](https://en.wikipedia.org/wiki/ISO_8601) (e.g. _2151-04-01_).
- `html-utf8` - HTML strings with UTF-8 characters (e.g. `Zürich <i class="icon"></i>`) 2.1.9.
- `html` - HTML strings (e.g. `<i>Tick</i>`).
- `string-utf8` - Plain text strings with UTF-8 characters (e.g. _Crème Brulée_). 2.1.0
- `string` - Plain text strings.

# Ajax

## Loading data

Ajax data is loaded by DataTables simply by using the [`ajax`](https://datatables.net/reference/option/ajax) option to set the URL for where the Ajax request should be made. For example, the following shows a minimal configuration with Ajax sourced data:

```js
$('#myTable').DataTable({
    ajax: '/api/myData'
});

```

## Data array location

1) Simple array of data:
```js
[
    {
        "name": "Tiger Nixon",
        "position": "System Architect",
        "salary": "$320,800",
        "start_date": "2011/04/25",
        "office": "Edinburgh",
        "extn": "5421"
    }
  
]


$('#myTable').DataTable({
    ajax: {
        url: '/api/myData',
        dataSrc: ''
    },
    columns: [
        // your column definitions here
    ]
});


```

2) Object with `data` property - note that the `data` parameter format shown here can be used with a simplified DataTables initialisation as `data` is the default property that DataTables looks for in the source data object.

```js
{
    "data": [
        {
            "name": "Tiger Nixon",
            "position": "System Architect",
            "salary": "$320,800",
            "start_date": "2011/04/25",
            "office": "Edinburgh",
            "extn": "5421"
        }
      
    ]
}

// Simple ajax call with default dataSrc = 'data'
$('#myTable').DataTable({
    ajax: '/api/myData',
    columns: [
        // your column definitions here
    ]
});

// Or explicitly setting dataSrc
$('#myTable').DataTable({
    ajax: {
        url: '/api/myData',
        dataSrc: 'data'
    },
    columns: [
        // your column definitions here
    ]
});


```


## Column data points

Now that DataTables knows where to get the data for the rows, we need to also tell it where to get the data for each cell in that row - this is done through the [`columns.data`](https://datatables.net/reference/option/columns.data) option.


1) Array of data - note that the array option does not require the [`columns.data`](https://datatables.net/reference/option/columns.data) option to be set. This is because the default value for [`columns.data`](https://datatables.net/reference/option/columns.data) is the column index (i.e. `0, 1, 2...`)

```js
[
    "Tiger Nixon",
    "System Architect",
    "$320,800",
    "2011/04/25",
    "Edinburgh",
    "5421"
]


$('#myTable').DataTable({
    ajax: 'your-data-source-url'
});

// or with column indexing for array data:

$('#myTable').DataTable({
    ajax: 'your-data-source-url',
    columns: [
        { data: 0 },
        { data: 1 },
        { data: 2 },
        { data: 3 },
        { data: 4 },
        { data: 5 }
    ]
});


```


2) Object of data:

```js
{
    "name": "Tiger Nixon",
    "position": "System Architect",
    "salary": "$320,800",
    "start_date": "2011/04/25",
    "office": "Edinburgh",
    "extn": "5421"
}


$('#myTable').DataTable({
    ajax: 'your-data-source-url',
    columns: [
        { data: 'name' },
        { data: 'position' },
        { data: 'salary' },
        { data: 'start_date' },  // fixed typo here
        { data: 'office' },
        { data: 'extn' }
    ]
});

```

3) Nested objects - in this case note that in the nested objects we use dotted object notation such as `hr.position` to access nested data. With this ability almost any JSON data structure can be used with DataTables:


```js

{
    "name": "Tiger Nixon",
    "hr": {
        "position": "System Architect",
        "salary": "$320,800",
        "start_date": "2011/04/25"
    },
    "contact": {
        "office": "Edinburgh",
        "extn": "5421"
    }
}


$('#myTable').DataTable({
    ajax: 'your-data-source-url',
    columns: [
        { data: 'name' },
        { data: 'hr.position' },
        { data: 'hr.salary' },
        { data: 'hr.start_date' },   // fixed typo here
        { data: 'contact.office' },
        { data: 'contact.extn' }
    ]
});

```


# Options

DataTables' [huge range of options](https://datatables.net/reference/option) can be used to customise the way that it will present its interface, and the features available, to the end user. This is done through its configuration options, which are set at initialisation time.


## Setting options

Configuration of DataTables is done by passing the options you want defined into the DataTables constructor as an object. For example:

## Setting options

```js
$('#example').DataTable({
    paging: false,
    scrollY: 400
});

```



# Server side Processing

Here’s a complete example of **DataTables server-side processing** using:

- **PostgreSQL** as the database
    
- **Node.js (Express)** as the backend server
    
- **jQuery DataTables** as the frontend
    
- **Sequelize** (optional) or raw SQL for PostgreSQL queries
    

---

## 🔧 1. Project Setup (Node.js + Express)

### Step 1: Initialize project

```bash
mkdir datatables-server-side
cd datatables-server-side
npm init -y
npm install express pg pg-hstore sequelize body-parser cors
```

---

## 🛠 2. Backend (Node.js + PostgreSQL)

### 📁 File structure

```
datatables-server-side/
├── server.js
├── config/
│   └── db.config.js
├── models/
│   └── index.js
│   └── user.model.js
└── routes/
    └── user.routes.js
```

---

### 📄 config/db.config.js

```js
module.exports = {
  HOST: "localhost",
  USER: "postgres",
  PASSWORD: "yourpassword",
  DB: "yourdatabase",
  dialect: "postgres",
  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000,
  },
};
```

---

### 📄 models/index.js

```js
const dbConfig = require("../config/db.config.js");
const { Sequelize, DataTypes } = require("sequelize");

const sequelize = new Sequelize(dbConfig.DB, dbConfig.USER, dbConfig.PASSWORD, {
  host: dbConfig.HOST,
  dialect: dbConfig.dialect,
  pool: dbConfig.pool,
});

const db = {};
db.Sequelize = Sequelize;
db.sequelize = sequelize;

db.users = require("./user.model.js")(sequelize, DataTypes);

module.exports = db;
```

---

### 📄 models/user.model.js

```js
module.exports = (sequelize, DataTypes) => {
  return sequelize.define("user", {
    id: {
      type: DataTypes.INTEGER,
      autoIncrement: true,
      primaryKey: true,
    },
    firstName: {
      type: DataTypes.STRING,
    },
    lastName: {
      type: DataTypes.STRING,
    },
    email: {
      type: DataTypes.STRING,
    },
  }, {
    timestamps: false,
  });
};
```

---

### 📄 routes/user.routes.js

```js
const express = require("express");
const router = express.Router();
const db = require("../models");
const User = db.users;
const { Op } = db.Sequelize;

router.post("/users", async (req, res) => {
  const draw = req.body.draw;
  const start = parseInt(req.body.start) || 0;
  const length = parseInt(req.body.length) || 10;

  const searchValue = req.body.search?.value;

  const whereClause = searchValue
    ? {
        [Op.or]: [
          { firstName: { [Op.iLike]: `%${searchValue}%` } },
          { lastName: { [Op.iLike]: `%${searchValue}%` } },
          { email: { [Op.iLike]: `%${searchValue}%` } },
        ],
      }
    : {};

  const totalRecords = await User.count();
  const filteredRecords = await User.count({ where: whereClause });

  const users = await User.findAll({
    where: whereClause,
    offset: start,
    limit: length,
  });

  res.json({
    draw,
    recordsTotal: totalRecords,
    recordsFiltered: filteredRecords,
    data: users,
  });
});

module.exports = router;
```

---

### 📄 server.js

```js
const express = require("express");
const bodyParser = require("body-parser");
const cors = require("cors");

const db = require("./models");
const app = express();
const PORT = 3000;

app.use(cors());
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

db.sequelize.sync({ force: false }).then(() => {
  console.log("DB connected.");
});

const userRoutes = require("./routes/user.routes");
app.use("/api", userRoutes);

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

---

## 🌐 3. Frontend (HTML + DataTables)

### 📄 index.html

```html
<!DOCTYPE html>
<html>
<head>
  <title>DataTables Server-Side with PostgreSQL</title>
  <link rel="stylesheet" href="https://cdn.datatables.net/1.13.6/css/jquery.dataTables.min.css">
</head>
<body>
  <h2>User List</h2>
  <table id="users" class="display" style="width:100%">
    <thead>
      <tr>
        <th>ID</th>
        <th>First Name</th>
        <th>Last Name</th>
        <th>Email</th>
      </tr>
    </thead>
  </table>

  <script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>
  <script src="https://cdn.datatables.net/1.13.6/js/jquery.dataTables.min.js"></script>

  <script>
    $(document).ready(function () {
      $('#users').DataTable({
        processing: true,
        serverSide: true,
        ajax: {
          url: 'http://localhost:3000/api/users',
          type: 'POST',
        },
        columns: [
          { data: 'id' },
          { data: 'firstName' },
          { data: 'lastName' },
          { data: 'email' },
        ],
      });
    });
  </script>
</body>
</html>
```

---

## 🧪 4. Test

- Ensure PostgreSQL is running and the `users` table has some data.
    
- Start server:
    

```bash
node server.js
```

- Open `index.html` in a browser (with a local web server or Live Server extension).
    
- You should see paginated, searchable user data fetched via server-side.
    

---

## ✅ Summary

- ✅ DataTables server-side pagination
    
- ✅ PostgreSQL backend
    
- ✅ Express API
    
- ✅ Sequelize ORM
    
- ✅ Search, pagination, draw support
    

---

Would you like this rewritten in **Python (Flask)** or **PHP** instead?

Excellent question — let's break this down clearly:

You're right in noticing that this code:

```js
const draw = req.body.draw;
const start = parseInt(req.body.start) || 0;
const length = parseInt(req.body.length) || 10;
const searchValue = req.body.search?.value;
```

**expects values in the request body**, but you're asking:

> "Where do these values come from, if we are just doing an `ajax: { type: 'POST' }`?"

---

## 🔁 The Key: DataTables Sends These Automatically

When you enable **server-side processing** in DataTables like this:

```js
$('#example').DataTable({
  serverSide: true,
  processing: true,
  ajax: {
    url: '/api/users',
    type: 'POST',
  },
  // ...
});
```

DataTables **automatically sends** a structured payload on every request, like:

```json
{
  "draw": 1,
  "columns": [
    { "data": "id", "searchable": true, "orderable": true, "search": { "value": "" } },
    { "data": "firstName", "searchable": true, "orderable": true, "search": { "value": "" } },
    ...
  ],
  "order": [
    { "column": 0, "dir": "asc" }
  ],
  "start": 0,
  "length": 10,
  "search": {
    "value": "john", // <- this is from the search box
    "regex": false
  }
}
```

---

## 🧠 So Where Do These Values Come From?

- `draw`: Internal counter used by DataTables to sync client/server state
    
- `start`: The **offset** (e.g., `0`, `10`, `20`...)
    
- `length`: The **number of rows per page**
    
- `search.value`: The **global search input** in the table
    

These values are sent in the **POST body** **automatically** by DataTables, so you don’t need to manually add them in your JS.

---

## ✅ What You Do Need to Do

You just need to ensure:

1. Your DataTable has `serverSide: true`
    
2. The `ajax` block has `type: 'POST'`
    
3. Your backend can parse the POST body (`body-parser` middleware or similar)
    

---

## 🛠 Make Sure You Have Body Parsing Middleware

If you’re using Express, make sure this is in your `server.js` or `app.js`:

```js
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
```

Otherwise, `req.body` will be `undefined`.

---

## ✅ Summary

- DataTables sends `draw`, `start`, `length`, `search`, `order`, and `columns` automatically in the request.
    
- You don't need to manually send these in your AJAX config.
    
- Your backend just needs to read them from `req.body`.
    

Let me know if you want to log or inspect a sample POST request from the browser to see the actual payload.