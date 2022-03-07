mysql -u root -p

# 13.1.4

We installed mySQL12, sequalize and express.js. Then we went through and set up all the folders and files. We added a schema.sql to the db file that simply creates a drops and creates a new database named just_tech_news_db and ran it with source b/schema.sql to build this new database. Finally we created a new file called connection.js in the config folder and pasted in the sequalize set up. The sequalize() asks for our database name, user name and password. This time instead of making that infomation visible we downloaded an npm package named dot env with npm install dotenv. Then be created a file in the root of our directory called .env and set these things to variables like this.

## Setting username and password to mine

DB_NAME='just_tech_news_db'
DB_USER='your-mysql-username'
DB_PW='your-mysql-password'
--.

Then we went in the .gitignore to make sure that the .env folder does not get pushed to gitHub.
