# 🚀 PHP Image Upload Project with Nginx, PHP 8.3, Appserver & DBserver + Amazon S3

### Fully DevOps-Ready PHP Image Upload System with S3 Storage and MySQL Database

**Author:** Prasad 👨‍💻  

---

## 🔥 Overview — What This Repo Delivers:-

- - Full PHP 8.3 + Nginx setup on EC2  
- - MariaDB/MySQL Database setup on separate instance  
- - Image upload with PHP to Amazon S3  
- - Secure, scalable, and production-ready  
- - Step-by-step setup with commands, Console, and CLI  
- - Troubleshooting and testing commands included  

You will find:

- - 📦 Architecture diagrams  
- - 🏗 Appserver + DBserver workflow  
- - 🛠 Step-by-step installation and setup  
- - 🔐 Security best practices  
- - 💡 Troubleshooting  
- - 🌟 Future enhancements  

---

## 🧭 1. Architecture (Simple View):-
```
           Users
             |
        EC2 AppServer
        /           \
PHP + Nginx S3   BucketDB Server
                     (MariaDB)

```

## 🔧 2. Appserver Setup (EC2):-

- Update packages:

```
sudo apt update
```
- Install Nginx, PHP, MariaDB client, PHP-FPM, PHP MySQL:
```
sudo apt install nginx mariadb-client php php8.3-fpm php8.3-mysql -y
```
- Restart services:
```
sudo service nginx restart
sudo service php8.3-fpm restart
```
- Navigate to web root:
```
cd /var/www/html/
```
- Create test.php:
```
sudo nano test.php
```
- Edit Nginx default site:
```
sudo nano /etc/nginx/sites-enabled/default
```
***Change in the Configer file as shown in the Image'***

- Reload and restart Nginx:
```
sudo service nginx reload
sudo service nginx restart
```
- Create form.html and upload.php:
```
sudo nano form.html
sudo nano upload.php
```
- Create uploads directory:
```
sudo mkdir uploads
sudo chmod -R 777 uploads/
```

## 🛠 3. Install AWS SDK for PHP:-

- Install Composer:
```
cd /var/www/html
```

```
sudo apt install zip -y
```

```
sudo curl -sS https://getcomposer.org/installer | sudo php
```

```
sudo mv composer.phar /usr/local/bin/composer
```

```
sudo ln -s /usr/local/bin/composer /usr/bin/composer
```
- Install AWS SDK:
```
sudo composer require aws/aws-sdk-php
```
- OR if issues:
```
sudo composer require aws/aws-sdk-php --ignore-platform-req=ext-curl --ignore-platform-req=ext-simplexml
```
## 🛠 4. Install AWS CLI:-
```
aws --version
```

```
sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

```
sudo apt install zip -y
```

```
sudo unzip awscliv2.zip
```

```
sudo ./aws/install --install-dir /usr/local/aws-cli --bin-dir /usr/local/bin
```

```
aws --version
```
## 🧱 5. DB Server Setup (MariaDB/MySQL):-

- Update and install MariaDB:
```
sudo apt update
sudo apt install mariadb-server -y
```
- Enter in Mysql:
```
sudo mysql
```
- Configure database and user:
```
ALTER USER 'root'@'localhost' IDENTIFIED BY '<Your-Password>';
```
- Create Database:
```
CREATE DATABASE facebook;
```
- Creare User:
```
CREATE USER 'user-name'@'<Your IP>' IDENTIFIED BY '<Your-Password>';
```
- Give Permissions to User:
```
GRANT ALL PRIVILEGES ON facebook.* TO 'user-name'@'<Your-IP>';
```

```
FLUSH PRIVILEGES;
```
- Enter In Database:
```
USE facebook;
```
- Create Table:
```
CREATE TABLE posts(id int PRIMARY KEY AUTO_INCREMENT, name VARCHAR(50), url VARCHAR(100));
```
- Exit from Mysql:
```
EXIT;
```
- Configure MariaDB for remote access:
```
cd /etc/mysql/mariadb.conf.d/
```

```
sudo nano 50-server.cnf
```
***Change in the Configure file as Shown in the Image add Your DBserver's privet IP***

- Reload & Restart Mariadb:
```
sudo service mariadb reload
sudo service mariadb restart
```
## 📝 6. HTML Form — form.html:-
```html
<!DOCTYPE html>
<html>
<body>
  <form action="upload.php" method="post" enctype="multipart/form-data">
    Name:<input type="text" id="name" name="name">
    Select image to upload:
    <input type="file" name="anyfile" id="anyfile">
    <input type="submit" value="Upload Image" name="submit">
  </form>
</body>
</html>
```
## 📝 7. PHP Upload Script — upload.php:-
```php

<?php
require 'vendor/autoload.php';
use Aws\S3\S3Client;

$s3Client = new S3Client([
    'version' => 'latest',
    'region'  => 'us-east-1'
]);

if($_SERVER["REQUEST_METHOD"] == "POST"){
    if(isset($_FILES["anyfile"]) && $_FILES["anyfile"]["error"] == 0){
        $allowed = array("jpg" => "image/jpg", "jpeg" => "image/jpeg", "gif" => "image/gif", "png" => "image/png");
        $filename = $_FILES["anyfile"]["name"];
        $filetype = $_FILES["anyfile"]["type"];
        $filesize = $_FILES["anyfile"]["size"];
        $ext = pathinfo($filename, PATHINFO_EXTENSION);

        if(!array_key_exists($ext, $allowed)) die("Error: Invalid file format.");
        if($filesize > 10*1024*1024) die("Error: File too large.");

        if(in_array($filetype, $allowed)){
            if(file_exists("uploads/" . $filename)){
                echo "$filename already exists.";
            } else {
                if(move_uploaded_file($_FILES["anyfile"]["tmp_name"], "uploads/" . $filename)){
                    $bucket = 'mybucket-19-11-25';
                    $file_Path = __DIR__ . '/uploads/'. $filename;
                    $key = basename($file_Path);

                    try {
                        $result = $s3Client->putObject([
                            'Bucket' => $bucket,
                            'Key'    => $key,
                            'Body'   => fopen($file_Path, 'r'),
                            'ACL'    => 'public-read'
                        ]);

                        echo "Image uploaded successfully. URL: ".$result->get('ObjectURL');
                        $urls3 = $result->get('ObjectURL');
                        $name = $_POST["name"];
                        $servername = "172.31.69.87";
                        $username = "swati";
                        $password = "Swati@123";
                        $dbname = "facebook";

                        $conn = mysqli_connect($servername, $username, $password, $dbname);
                        if (!$conn) die("Connection failed: " . mysqli_connect_error());

                        $sql = "INSERT INTO posts(name,url) VALUES('$name','$urls3')";
                        if(mysqli_query($conn, $sql)) echo "New record created successfully";
                        else echo "Error: " . $sql . "<br>" . mysqli_error($conn);
                        mysqli_close($conn);

                    } catch (Aws\S3\Exception\S3Exception $e){
                        echo "Error uploading file: ".$e->getMessage();
                    }
                } else echo "File not uploaded.";
            }
        }
    } else echo "Upload error: " . $_FILES["anyfile"]["error"];
}
?>
```
## 🔐 8. Security & Permissions:-

Ensure uploads/ folder has correct permissions:
```
sudo chmod -R 777 uploads/
```
- Do not expose DB credentials in production
- Use IAM Roles for S3 access if possible

## 🔥9. Steps to Allow App Server to Connect to DB :-

- Follow these steps to allow AppServer → DBServer (MariaDB/MySQL) traffic on port 3306.

### 🛠 1. Open EC2 Dashboard

- Go to AWS Console
- Open EC2 Service
- From left menu → click Security Groups

### 🛠 2. Select DB-Server Security Group

- Search for the DB Server SG
- Click on it to open details

### 🛠 3. Add Inbound Rule

- Go to Inbound Rules
- Click Edit inbound rules
- Click Add rule

### 🛠 4. Configure the Rule

- Type → MySQL/Aurora
- Port → 3306
- Source → App-Server Security Group
- Click inside Source field
- Select Security Group
- Choose your AppServer-SG
- Description → Allow DB access from App Server

### 🛠 5. Save the Rule

- Click Save rules
- Inbound rule is now active

## 🟢 Now What Happens?

- DB Server allows traffic only from AppServer
- AppServer can run PHP + DB queries safely
- Prevents internet or unknown IPs from accessing DB
- Improves security and reduces attack surface

## 🛠 10. Testing:-

- Test form upload via browser
- Check S3 bucket for uploaded images
- Check database for inserted records:
```
SELECT * FROM posts;
```
## 📁 11. Useful Commands Summary:-

- 1] sudo service nginx restart
- 2] sudo service php8.3-fpm restart
- 3] sudo mkdir uploads && chmod -R 777 uploads/
- 4] aws s3 ls
- 5] aws s3 cp localfile s3://mybucket/

## 🌟 12. Future Enhancements:-
- Add image thumbnail generation
- Add CloudFront + CDN for images
- Add signed URLs for private content
- Convert project to Docker + ECS/EKS deployment

## ✍️ 13. Author:-

Prasad 👨‍💻

Cloud • DevOps • AWS • PHP • Full-stack

⭐ If you like this repo, drop a star on GitHub!

## 📩 Connect With Me :
If you’d like to collaborate, discuss projects, or just say hello — feel free to reach out!  

### 🔗 Social & Professional Links:

- 🌐 [Portfolio Website](https://prasad-bhoite19.github.io/prasad-portfolio/)  
- 💼 [LinkedIn](http://linkedin.com/in/prasad-bhoite-a38a64223)  
- 🐙 [GitHub](https://github.com/Prasad-bhoite19)  
- ✉️ [Email](prasadsb2002@gmail.com)  

💬 Always open for opportunities in **Cloud, DevOps, and Full-Stack Projects**
