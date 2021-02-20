**Artifact Three**

**(Demonstrates Databases)**

For this artifact, I created a JavaScript program that can access a remotely hosted MySQL database. The purpose of this program is to prompt the user to create, read, update, or delete a table, or query the database. If the user picks one of the first four options, they will be guided through the process of that particular selection, for example, in the create table option, the user is prompted to give the new table a name, give a new column in the table a name, decide if the column should be an INT, CHAR, or DATE data type, and lastly if the column should allow NULL data. This system allows the user to create, read, update, and delete tables from a database without having to know any of the SQL language. Lastly, the query database option allows the user to type a freehand query to the database, allowing them to perform more technical searches or other actions that would be too complicated to program individually the way the other four options were done.

**Code**

SQL-UI.js
```javascript
const mysql = require('mysql');

var config =
{
    host: 'example.mysql.database.azure.com', //Enter database host url
    user: 'exampleUsername', //Enter database login username
    password: 'examplePassword', //Enter database login password
    database: 'exampleTable', //Enter database table to modify
    port: 3306, //Enter database port number (Default 3306)
    ssl: true
};

const conn = new mysql.createConnection(config);
var readlineSync = require('readline-sync'); //Required to read user input

var tableFields = [];
var maxTableFields = 48; //Total number of columns allowed to be added when making a new table * 3 (ex. 8 columns would equal 24 maxTableFields_.
var currentColumn = 1;
var resultsLimit = 0;
var userChoice = '';
var tableName = '';
var modifyChoice = '';

conn.connect( //Connect to mySQL server
    function (err) { 
    if (err) { 
        console.log("!!! Cannot Connect !!! Error:");
        throw err;
    }
    else
    {
		console.log("Connection established.");
		
			console.log('');
			console.log('Choose action:');
			console.log('Create Table [c]');
			console.log('Read   Table [r]');
			console.log('Update Table [u]');
			console.log('Delete Table [d]');
			console.log('Query  Database [q]');
			console.log('Exit [e]');
			userChoice = readlineSync.question('');
			console.log(`You selected ${userChoice}`);
			
			switch (userChoice) {
				case 'c':
					createTable();
					break;
				case 'r':
					readTable();
					break;
				case 'u':
					updateTable();
					break;
				case 'd':
					deleteTable();
					break;
				case 'q':
					queryDatabase();
					break;
				case 'e':
					break;
				default:
					console.log('Invalid selection');
			};
		   
			conn.end(function (err) { 
			if (err) {
				throw err;
			} else  {
				console.log('Disconnecting') 
			};
			});
    };
});

function createTable(){ //Create a new table
	tableName = readlineSync.question('Input table name (Overrides table with identical name): ');
	console.log(`tableName = ${tableName}`);
	console.log('Primary Key created');
	
	
	for (var i = 0;  i < maxTableFields; i += 3) {
		tableFields[i] = readlineSync.question(`Input column #${currentColumn} name (Max columns = ${maxTableFields/3}) (Type "exit" to end): `);
		if (tableFields[i] == 'exit') { //Let user keep making more columns until they type "exit"
			break;
		};
		
		tableFields[i+1] = readlineSync.question(`Input column #${currentColumn} data type ("INT", "CHAR", or "DATE": `); //Select column data type, keep asking until valid selection
		while (tableFields[i+1] != 'INT' && tableFields[i+1] != 'CHAR' && tableFields[i+1] != 'DATE') {
			console.log('Invalid data type');
			tableFields[i+1] = readlineSync.question(`Input column #${i} data type ("INT", "CHAR", or "DATE": `);
		};
		
		tableFields[i+2] = readlineSync.question('Allow NULL ("y", "n")": '); //Select if column can be NULL, keep asking until valid selection
		while (tableFields[i+2] != 'y' && tableFields[i+2] != 'n') {
			console.log('Invalid choice');
			tableFields[i+2] = readlineSync.question('Allow NULL ("y", "n")": ');
		};
		
		if (tableFields[i+2] == 'y') { //If allow NULL, tablefields = ' ', else tableFields = ' NOT '
			tableFields[i+2] = ' ';
		} else {
			tableFields[i+2] = ' NOT ';
		};
		currentColumn += 1;
	};
	
    conn.query(`DROP TABLE IF EXISTS ${tableName};`)
		conn.query(`CREATE TABLE ${tableName} (${tableName}_id INT(8) UNSIGNED AUTO_INCREMENT PRIMARY KEY);`) //Create Primary Key
		
		for (var i = 0; i < tableFields.length - 1; i += 3) {
			if (tableFields[i+1] == 'DATE') { //DATE columns require special treatment
				conn.query(`ALTER TABLE ${tableName} ADD COLUMN ${tableFields[i]} ${tableFields[i+1]}${tableFields[i+2]}NULL;`)
			} else {
				conn.query(`ALTER TABLE ${tableName} ADD COLUMN ${tableFields[i]} ${tableFields[i+1]}(64)${tableFields[i+2]}NULL`)
			};
			
		};
		console.log(`Table ${tableName} successfully created`);
};

function readTable(){ //Read all entries in the selected table
	tableName = readlineSync.question('Select table: ');
	tableFields[0] = readlineSync.question('Input column to sort by: ');
	resultsLimit = readlineSync.question('Limit results to n results (0 = No Limit): ');
	
	if (resultsLimit == 0) {
		conn.query(`SELECT * FROM ${tableName} ORDER BY ${tableFields[0]}`, function (err, results, fields) {
        if (err) throw err;
        for (i = 0; i < results.length; i++) {
			console.log('Row: ' + JSON.stringify(results[i]));
        }
    })
	} else {
		conn.query(`SELECT * FROM ${tableName} ORDER BY ${tableFields[0]} LIMIT ${resultsLimit}`, function (err, results, fields) {
		if (err) throw err;
		for (i = 0; i < results.length; i++) {
			console.log('Row: ' + JSON.stringify(results[i]));
		}
	})
	};
}

function updateTable(){ //Add, Drop, or Modify a column in the selected table
	tableName = readlineSync.question('Select table: ');
	modifyChoice = readlineSync.question('Input modification ("ADD", "DROP", "MODIFY": ');
	
	while (modifyChoice != 'ADD' && modifyChoice != 'DROP' && modifyChoice != 'MODIFY') {
			console.log('Invalid choice');
			modifyChoice = readlineSync.question('Input modification ("ADD", "DROP", "MODIFY": ');
		};
	
	tableFields[0] = readlineSync.question('Input column name: ');
	
	if (modifyChoice == 'ADD' || modifyChoice == 'MODIFY') {
		tableFields[1] = readlineSync.question('Input column data type ("INT", "CHAR", or "DATE": ');
		tableFields[2] = readlineSync.question('Allow NULL ("y", "n")": ');
		
		if (tableFields[2] == 'y') { //If allow NULL, tablefields = ' ', else tableFields = ' NOT '
			tableFields[2] = ' ';
		} else {
			tableFields[2] = ' NOT ';
		};
		
		if (tableFields[1] == 'DATE') {
			conn.query(`ALTER TABLE ${tableName} ${modifyChoice} COLUMN ${tableFields[0]} ${tableFields[1]}${tableFields[2]}NULL;`)
		} else {
			conn.query(`ALTER TABLE ${tableName} ${modifyChoice} COLUMN ${tableFields[0]} ${tableFields[1]}(64)${tableFields[2]}NULL`)
		};
	};
	
	if (modifyChoice == 'DROP') {
		conn.query(`ALTER TABLE ${tableName} DROP COLUMN ${tableFields[0]};`)
	}
};

function deleteTable(){ //Drop the selected table
    tableName = readlineSync.question('Select table: ');
	conn.query(`DROP TABLE IF EXISTS ${tableName}`)
	console.log(`Table ${tableName} successfully deleted`);
};

function queryDatabase(){ //Create a freehand SQL query
	tableName = readlineSync.question('Enter SQL query: ');
	conn.query(`${tableName}`, function (err, results, fields) {
        if (err) throw err;
        for (i = 0; i < results.length; i++) {
			console.log('Row: ' + JSON.stringify(results[i]));
        }
    })
};
```

**Narrative**

I chose to include this enhancement because of the example enhancements presented in my CS 499: Computer Science Capstone course, this one sounded the most interesting to me. I thought it would be a nice challenge to create a project in JavaScript as I have no formal training iwith the language, though in the end it turned out similar to Java. By creating this enhancement, I have showcased my abilities to work with JavaScript and MySQL, two languages that are widely used in the professional world.

When I initially planned to create this enhancement, I believed I could create it using JavaScript almost exclusively, though what I didn’t know at the time was that JavaScript must run in a web browser and is entirely client-side, making it impossible to create a stand-alone program that performs server-side SQL queries with it by itself. Perhaps the biggest challenge I faced creating this enhancement was figuring out what tools I would need to use to start making it. Many posts online recommended using a mix of JavaScript, HTML, and PHP, but having to learn how to write code for HTML and PHP, both of which I have no experience with, was too daunting. Eventually, I did end up finding a solution, Node.js, a standalone version of JavaScript that offered enough functionality to let me create a standalone program that could connect to and query a MySQL database hosted on Microsoft Azure. Once I had everything installed and successfully ran a test program to confirm I could interact with my SQL database, I then began tinkering with example code offered by Microsoft until I eventually figured out how to take user input and run queries on the database exclusively through the JavaScript program. At this point progress became smoother, as I know enough about programming in general to create a program with the functionality that I needed.

To run the program, the computer needs to have Node.js, the MySQL driver, and a Node.js library called “readline-sync” installed. A simple guide to downloading Node.js and the MySQL driver can be found on “https://docs.microsoft.com/en-us/azure/mysql/connect-nodejs”, then afterwards the Node.js library can be downloaded with the cmd command “npm install readline-sync”.

**Artifacts**

[Home Page](./Home.md)

[Software Design and Engineering](./ArtifactOne.md)

[Algorithms and Data Structures](./ArtifactTwo.md)
