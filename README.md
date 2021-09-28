Create EC2 Instance

aws ec2 run-instances --image-id ami-09e67e426f25ce0d7 --count 1 --instance-type t2.micro --key-name xavier --security-group-ids sg-ca905e86 --subnet-id subnet-9f9b1ff8
_______________
 
Upgrade process and install packages

sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt-get install mysql-client apache2 php php-mysql -y
sudo systemctl restart apache2.service

_______________
Create RDS 
aws rds create-db-instance \
    --db-instance-identifier test-mysql-instance \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --master-username admin \
    --master-user-password xxxxxxxx\
    --allocated-storage 8

_______________
Create the application

sudo vi /var/www/html/form.html

<head>
<title>
Test Page
</title>
</head>
<body>
<form action="form_submit.php" class="alt" method="POST">
<div class="row uniform">

	<div class="name">
		<input name="name" id="" placeholder="name" type="text">
	</div>

	<div class="owner">
		<input name="owner" id="" placeholder="owner" type="text">
	</div>

	<div class="species">
		<input name="species" id="" placeholder="species" type="text">
	</div>

	<div class="sex">
		<input name="sex" id="" placeholder="sex" type="text">
	</div>

</div>
<br/>
<input class="alt" value="Submit" name="submit" type="submit">
</form>
</body>

_______________
Create the database

mysql -h test-mysql-instance.clydyuazbnns.us-east-1.rds.amazonaws.com -P 3306 -u admin -p

create database dev_to;
use dev_to;
create table pet(
    id int NOT NULL AUTO_INCREMENT,
    name varchar(20),
    owner varchar(20),
    species varchar(20),
	sex varchar(1),
    PRIMARY KEY (id)
); 

_______________
Continuing creating the application

sudo vi /var/www/html/form_submit.php

<?php
$host = "test-mysql-instance.clydyuazbnns.us-east-1.rds.amazonaws.com";
$db_name = "dev_to";
$username = "admin";
$password = "xxxxxxxx";
$connection = null;
try{
$connection = new PDO("mysql:host=" . $host . ";dbname=" . $db_name, $username, $password);
$connection->exec("set names utf8");
}catch(PDOException $exception){
echo "Connection error: " . $exception->getMessage();
}

function saveData($name, $owner, $species, $sex){
global $connection;
$query = "INSERT INTO pet(name, owner, species, sex) VALUES( :name, :owner, :species, :sex)";

$callToDb = $connection->prepare( $query );
$name=htmlspecialchars(strip_tags($name));
$owner=htmlspecialchars(strip_tags($owner));
$speciesname=htmlspecialchars(strip_tags($species));
$sex=htmlspecialchars(strip_tags($sex));
$callToDb->bindParam(":name",$name);
$callToDb->bindParam(":owner",$owner);
$callToDb->bindParam(":species",$species);
$callToDb->bindParam(":sex",$sex);

if($callToDb->execute()){
return '<h3 style="text-align:center;">We will get back to you very shortly!</h3>';
}
}

if( isset($_POST['submit'])){
$name = htmlentities($_POST['name']);
$owner = htmlentities($_POST['owner']);
$species = htmlentities($_POST['species']);
$sex = htmlentities($_POST['sex']);

//then you can use them in a PHP function. 
$result = saveData($name, $owner, $species, $sex);
echo $result;
}
else{
echo '<h3 style="text-align:center;">A very detailed error message ( ͡° ͜ʖ ͡°)</h3>';
}
?>

_______________
Insert values

http://host/form.html 

_______________
Continuing creating the application

sudo vi /var/www/html/form_query.php

<?php
$host    = "test-mysql-instance.clydyuazbnns.us-east-1.rds.amazonaws.com";
$user    = "admin";
$pass    = "secret99";
$db_name = "dev_to";

//create connection
$connection = mysqli_connect($host, $user, $pass, $db_name);

//test if connection failed
if(mysqli_connect_errno()){
    die("connection failed: "
        . mysqli_connect_error()
        . " (" . mysqli_connect_errno()
        . ")");
}

//get results from database
$result = mysqli_query($connection,"SELECT * FROM pet");
$all_property = array();  //declare an array for saving property

//showing property
echo '<table class="data-table">
        <tr class="data-heading">';  //initialize table tag
while ($property = mysqli_fetch_field($result)) {
    echo '<td>' . $property->name . '</td>';  //get field name for header
    array_push($all_property, $property->name);  //save those to array
}
echo '</tr>'; //end tr tag

//showing all data
while ($row = mysqli_fetch_array($result)) {
    echo "<tr>";
    foreach ($all_property as $item) {
        echo '<td>' . $row[$item] . '</td>'; //get items using property value
    }
    echo '</tr>';
}
echo "</table>";
?>

_______________
Insert values
http://18.234.139.143/form.html 

Show values inserted
http://18.234.139.143/form_query.php
