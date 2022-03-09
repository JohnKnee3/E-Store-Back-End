mysql -u root -p

# 13.1.4

We installed mySQL12, sequalize and express.js. Then we went through and set up all the folders and files. We added a schema.sql to the db file that simply creates a drops and creates a new database named just_tech_news_db and ran it with source b/schema.sql to build this new database. Finally we created a new file called connection.js in the config folder and pasted in the sequalize set up. The sequalize() asks for our database name, user name and password. This time instead of making that infomation visible we downloaded an npm package named dot env with npm install dotenv. Then be created a file in the root of our directory called .env and set these things to variables like this.

## Setting username and password to mine

DB_NAME='just_tech_news_db'
DB_USER='your-mysql-username'
DB_PW='your-mysql-password'
--.

Then we went in the .gitignore to make sure that the .env folder does not get pushed to gitHub.

# 13.1.5

Created our first model using sequelize. We made the User.js file and copy and pasted in the info to start it's table.

## At the top we required

const { Model, DataTypes } = require("sequelize");
const sequelize = require("../config/connection");

// create our User model
class User extends Model {}
--.
The first one grabs the Model class from sequelize, then we require our SQL connection. Afterwards we make a new class called USER and give it all the functionality of the Model class we brought in from sequelize.

## Then we ran User.it() and made it look like this.

// define table columns and configuration
User.init(
{
// define an id column
id: {
// use the special Sequelize DataTypes object provide what type of data it is
type: DataTypes.INTEGER,
// this is the equivalent of SQL's `NOT NULL` option
allowNull: false,
// instruct that this is the Primary Key
primaryKey: true,
// turn on auto increment
autoIncrement: true,
},
// define a username column
username: {
type: DataTypes.STRING,
allowNull: false,
},
// define an email column
email: {
type: DataTypes.STRING,
allowNull: false,
// there cannot be any duplicate email values in this table
unique: true,
// if allowNull is set to false, we can run our data through validators before creating the table data
validate: {
isEmail: true,
},
},
// define a password column
password: {
type: DataTypes.STRING,
allowNull: false,
validate: {
// this means the password must be at least four characters long
len: [4],
},
},
},
{
sequelize,
timestamps: false,
freezeTableName: true,
underscored: true,
modelName: "user",
}
);
--.
In here the first object we created holds all the columns and defines each one one at a time giving the sequelize syntax so it can talk to mySQL and create the parameters for these columns. The second object configures certain options for the entire table.

Lastly we export the user with madule.exports = User; and went and made an index.js in the models folder that requires this User.js and then uses module.exports = {User}; to future proof the app so we can be ready for multiple exports.

# 13.1.6

Pretty big page. The gist is we added the get all, get by id, post, put and delete to the user-routes.js. Then we created an index.js in the routes/api folder, then we created another index in the routes folder. Each of these talk to each other and append to the URL in order.

## Finally we got our server,js up and running like this.

const express = require('express');
const routes = require('./routes');
const sequelize = require('./config/connection');

const app = express();
const PORT = process.env.PORT || 3001;

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// turn on routes
app.use(routes);

// turn on connection to db and server
sequelize.sync({ force: false }).then(() => {
app.listen(PORT, () => console.log('Now listening'));
});
--.
We required the routes folder which will work because of the two index files which serve as a hub to talk to everything. Then we used app.use(routes); to turn them on. Then we started the sequalize at the bottom and are using force false for now. It claimed that if it was true it would drop and create the db each time we load and that this will counter that.

# 13.2.4

We downloaded bcrypt to protect our passwords using npm install bcrypt and requiring at the top of the User.js file that creates the rules for the User table. We are also planning on adding async to this because running this package is intensive and could take time, making it asynchronous will allow the function to continue while the bcrypt hash does it's thing.

# 13.2.5

Added code to the second object in the User,js which controls the table settings to wait for the passwords to hash out the brcypt when we create an account and when we update the account.

## The added hooks looks like this in the second object

{
hooks: {
// set up beforeCreate lifecycle "hook" functionality
async beforeCreate(newUserData) {
newUserData.password = await bcrypt.hash(newUserData.password, 10);
return newUserData;
},
// set up beforeUpdate lifecycle "hook" functionality
async beforeUpdate(updatedUserData) {
updatedUserData.password = await bcrypt.hash(
updatedUserData.password,
10
);
return updatedUserData;
},
},
sequelize,
timestamps: false,
freezeTableName: true,
underscored: true,
modelName: "user",
}
--.

Finally we had to slide into the user-routes.js and tweak with the put. The create was fine but in the put we needed to add a line

## just before the where in the put.

User.update(req.body, {
individualHooks: true,
where: {
id: req.params.id,
},
})

# 13.2.6

Created the post that verifies the user name and password.

## In user-routes.js it looks like this

router.post("/login", (req, res) => {
// expects {email: 'lernantino@gmail.com', password: 'password1234'}
User.findOne({
where: {
email: req.body.email,
},
}).then((dbUserData) => {
if (!dbUserData) {
res.status(400).json({ message: "No user with that email address!" });
return;
}

    const validPassword = dbUserData.checkPassword(req.body.password);
    if (!validPassword) {
      res.status(400).json({ message: "Incorrect password!" });
      return;
    }

    res.json({ user: dbUserData, message: "You are now logged in!" });

});
});
--.
First we set up the URL to perform a post with /login. Afterwards we use teh sequelize findOne to look things up by email this time. If it finds a matching email address everything is fine and just to the checkPassword function that passes in the current password.

## This function is on the User.js page and looks like this

class User extends Model {
// set up method to run on instance data (per user) to check password
checkPassword(loginPw) {
return bcrypt.compareSync(loginPw, this.password);
}
}
--.

Then over here bycrypt will work it's magic to compare the typed in plain text password to it's post hashed password. If it matched it sends back a true and you are good to log in.

# 13.3.4

Created the Post table in the Post.js file in the Models folder. Then we went into the index.js in the models folder and required and exported it. The main new thing about this post table is that it uses a foreign key that references the User table .

## The code for the table looks like this.

Post.init(
{
id: {
type: DataTypes.INTEGER,
allowNull: false,
primaryKey: true,
autoIncrement: true,
},
title: {
type: DataTypes.STRING,
allowNull: false,
},
post_url: {
type: DataTypes.STRING,
allowNull: false,
validate: {
isURL: true,
},
},
user_id: {
type: DataTypes.INTEGER,
references: {
model: "user",
key: "id",
},
},
},
{
sequelize,
freezeTableName: true,
underscored: true,
modelName: "post",
}
);
--.

Here we have the new guy user_id that is using the references:. Within that it targets the table by referencing model by with the table's name of user. Finally we use the key: which both simultaneously is our foreign key and references their primary key, we also target it by calling the foreign key by it's name of id as it is named in the User's table.

# 13.3.5

Went into the models index.js and had to define the relationship between the Post and User table so sequelize can undertand it.

## The code to define their relationship looked like this.

// create associations
User.hasMany(Post, {
foreignKey: "user_id",
});

Post.belongsTo(User, {
foreignKey: "user_id",
});
--.

This reads pretty well but to sum it up it says that the User table can have multiple post tables associated with it. Then we in and let it know that Post can only belong to ONE user. Both times we reference the fk which is user_id.
