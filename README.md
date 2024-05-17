# Solution of Query string attack (Sorgu dizisi saldırısı Çözümü)
Bu saldırıyı [anlatım sayfası](https://github.com/yasir723/query-string-attack)nda gördüğümüz gibi, hackerlar olarak URL'deki parametreleri kullanarak hassas bilgileri ele geçirip oturum çalabildiğimizi öğrendik. Bu sayfada, geliştirici olarak bu tür saldırılardan sistemimizi nasıl koruyabileceğimizi öğreneceğiz. Öncelikle, hiçbir durumda hassas bilgileri URL ile göndermemeliyiz ki bu ciddi bir güvenlik açığı oluşturur. İleri saldırılarda, hassas bilgilerin güvenli bir şekilde nasıl gönderilmesi gerektiği anlatılmıştır. Ancak bu sayfada, eğer hassas bilgileri göndermemiz gerekirse, bunu nasıl güvenli hale getirebileceğimiz üzerinde duracağız.


Giriş yapıp hareketler sayfasına gittiğimizde urlde userName parametresini gönderiyoruz bu aşağıdaki fotoğrafta göreceğiz:

<div align="center">
    <img src="https://github.com/yasir723/solution-of-query-string-attack/assets/111686779/8830758a-edfc-4459-89b5-119a5b5af204">
</div>

ancak userName genellikle kolay tahmin edilebilen bir terimdir bu yüzden kullanıcıya ait userToken gibi zor tahmin edilebilen bir terim parametre olarak göndermemiz gerek, ardından bu parametreyi sunucu tarafında kullanarak kullanıcının userName özelliğini ulaşırız.

UserToken: genellikle kullanıcı oturumlarını yönetmek için kullanılan bir kimlik belgesidir. Rasgele oluşturulur ve kullanıcının kimliğini doğrulamak için sunucu tarafından kullanılır.
<div align="center">
    <img src="https://github.com/yasir723/solution-of-query-string-attack/assets/111686779/71eeb696-de95-4f33-b1fd-8d18bea3b766">
</div>

veritabanındaki kullanıcı tablosuna bakarasak bu şekilde göreceğiz:
<div align="center">
    <img src="https://github.com/yasir723/solution-of-query-string-attack/assets/111686779/8d46b241-825c-4941-8740-3b7e674b8473">
</div>


işte userToken gibi bilgisi zor tahmin edilen bir terim olduğu için bu şekilde kullanarabiliriz ancak güvenlik açısından en iyi yöntem değildir. kullanıcı kimliğini doğrulamak için kullandığımız sunucu tarafındaki kod `login.php sayfası` bu şekilde username parametre olarak gönderiyoruz:
```php
<?php
//Database Authentication
require("DBInfo.php");

// Server side code
//Read form submit info post request
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $userName = $_POST['userName'];
    $password = $_POST['password'];
}
//Not secure post call
if(!empty($userName) and !empty($password)) {
    //connect to database
    $connect = mysqli_connect($hostDB, $userDB,$passwordDB,$databaseDB);
    if(mysqli_connect_errno()){
        die(" cannot connect to database ". mysqli_connect_error());
    }

    $query ="select * from login  where userName='" .
        $userName ."' and password='" . $password ."'" ;

    $result= mysqli_query($connect,$query);
    if (!$result){
        die(' Error cannot run query');
    }

    $loginInUser=null;
    while ($row= mysqli_fetch_assoc($result)) {
        $loginInUser= $row["userName"];
        break; // to be save
    }

    mysqli_free_result($result);
    mysqli_close($connect);

    echo "<pre style=\"background-color: #DEDAF1; font-weight: bold;\">";
    echo "Server data:</br>";
    echo "Kullanıcı Adı: " . $userName. "</br>";
    echo "Şifre: " . $password. "</br>";
    echo "query: " . $query . "</br>";
    if (! empty($loginInUser)) {

       // $_SESSION["userName"] = $loginInUser;

        echo "</br><div class=\"alert\" style=\"background-color:#68479d; color: white;font-weight: bold;\" >Veritabanı Bildirimi: Giriş Başarılı - Kullanıcı Adı: (". $loginInUser .")";
        echo "</br></br><a class=\"btn \" style=\"background-color:#280F4D; color: white;font-weight: bold;\" href='myActivities.php?userName=" .$loginInUser ."'> Hareketlerim</a>";
         echo "</div>";

    }else{
        echo "</br><div class=\"alert alert-danger\">Database Message: Fail login</div>";
    }
    echo "</pre>";
}

?>
```

Ancak biz userName yerine userToken göndermek istiyoruz bu yüzden userName yazılan yerlere userToken yazacağız
ilk adım olarak giriş yapmış olan kullanıcının userName bilgisi değil, userToken bilgisini bir değişkene atarız
```php
$userToken=null;
while ($row= mysqli_fetch_assoc($result)) {
    $userToken= $row["userToken"];
    break; // to be save
}
```
2. adım olarak başarılı bir giriş yapıldıysa ayrı bir değişkene atadığımız userToken bilgisini `Hareketlerim` butonuna tıklandığında kullanıcı yönlendirilecek linkte parametre olarak gönderilen bilgilerde düzeltmemiz gerekiyor, userName yerine userToken göndermemiz gerek.
```php
if (!empty($userToken)) {
    echo "</br><div class=\"alert\" style=\"background-color:#68479d; color: white;font-weight: bold;\" >Veritabanı Bildirimi: Giriş Başarılı - Kullanıcı Adı: (" . $userName . ")";
    echo "</br></br><a class=\"btn \" style=\"background-color:#280F4D; color: white;font-weight: bold;\" href='myActivities.php?userName=" . $userToken . "'> Hareketlerim</a>";
    echo "</div>";
} else {
    echo "</br><div class=\"alert alert-danger\">Database Message: Fail login</div>";
}
echo "</pre>";
```

`login.php` sayfası güncel kodu bu şekilde olacaktır:
```php
<?php
//Database Authentication
require("DBInfo.php");

// Server side code
//Read form submit info post request
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $userName = $_POST['userName'];
    $password = $_POST['password'];
}
//Not secure post call
if (!empty($userName) and !empty($password)) {
    //connect to database
    $connect = mysqli_connect($hostDB, $userDB, $passwordDB, $databaseDB);
    if (mysqli_connect_errno()) {
        die(" cannot connect to database " . mysqli_connect_error());
    }

    $query = "select * from login  where userName='" .
        $userName . "' and password='" . $password . "'";

    $result = mysqli_query($connect, $query);
    if (!$result) {
        die(' Error cannot run query');
    }

    $userToken = null;
    while ($row = mysqli_fetch_assoc($result)) {
        $userToken = $row["userToken"];
        break; // to be save
    }

    mysqli_free_result($result);
    mysqli_close($connect);

    echo "<pre style=\"background-color: #DEDAF1; font-weight: bold;\">";
    echo "Server data:</br>";
    echo "Kullanıcı Adı: " . $userName . "</br>";
    echo "Şifre: " . $password . "</br>";
    echo "query: " . $query . "</br>";
    if (!empty($userToken)) {
        echo "</br><div class=\"alert\" style=\"background-color:#68479d; color: white;font-weight: bold;\" >Veritabanı Bildirimi: Giriş Başarılı - Kullanıcı Adı: (" . $userName . ")";
        echo "</br></br><a class=\"btn \" style=\"background-color:#280F4D; color: white;font-weight: bold;\" href='myActivities.php?userToken=" . $userToken . "'> Hareketlerim</a>";
        echo "</div>";
    } else {
        echo "</br><div class=\"alert alert-danger\">Database Message: Fail login</div>";
    }
    echo "</pre>";
}
?>
```


Böylece kullanıcı `Hareketlerim` butonuna tıkladığı zaman gönderilecek bilgi userName yerine userToken gönderilecektir. Şimdi bu bilgiyi kullanacak kod `myActivities.php` userName alan yerinde userToken almasını yazmamız gerek, güncellenecek kod bu şekilde:

```php
<?php require 'headerTab.php' ?>

<body style="font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;">
    <div class="container" style="width: 50%; margin-left:auto;margin-right: auto;font-size: 15px;">
        </br>
        </br>
        </br>
        <form id='login' action="myActivities.php?userName=<?= $_GET['userName']; ?>" method='post' accept-charset='UTF-8'>
            <div class="panel">
                <div class="panel-heading" style="background-color:#280f4d; color:#fff">Para Gönderme</div>
                <div class="panel-body">
                    <div class="form-group">
                        <label for='ToUserName'>Alıcının Kullanıcı Adı:</label>
                        <input type='text' name='ToUserName' class="form-control" id='ToUserName' maxlength="50" required />
                        <label for='Amount'>Miktar:</label>
                        <input type='text' name='Amount' class="form-control" id='Amount' maxlength="50" required />
                        <input type="hidden" name="fromUserName" value="<?= $_GET['userName']; ?>" />

                        <input type='submit' style="font-size: 20px; padding-left:15px; padding-right:15px; background-color:#280f4d; color:#fff" class="btn btn-success" name='Submit' id='submit' value='Gönder' />
                    </div>
                </div>
            </div>

        </form>

        <h1> Hareketler</h1>
        <table class="table table-bordered">
            <thead>
                <tr>
                    <td> Transfer Anahtarı </td>
                    <td> Gönderici </td>
                    <td> Alıcı</td>
                    <td> Miktar</td>
                </tr>
            </thead>
            <tbody>



                <?php
                //Database Authentication
                require("DBInfo.php");

                $userName = $_GET['userName'];

                //connect to database
                $connect = mysqli_connect($hostDB, $userDB, $passwordDB, $databaseDB);
                if (mysqli_connect_errno()) {
                    die(" cannot connect to database " . mysqli_connect_error());
                }


                //Add new Activitiy
                if (!empty($_POST['fromUserName']) and !empty($_POST['ToUserName'])) {

                    $query = "insert into activities(fromUserName,ToUserName,Amount)
                    values ('" . $_POST['fromUserName'] . "','" . $_POST['ToUserName'] . "'," . $_POST['Amount'] . ")";

                    $result = mysqli_query($connect, $query);
                    if (!$result) {
                        die(' Error cannot run query');
                    }
                }

                // get user activities
                if (!empty($userName)) {
                    $query = "select * from activities  where fromUserName='" . $userName . "' or ToUserName='" . $userName . "'";
                    $result = mysqli_query($connect, $query);
                    if (!$result) {
                        die(' Error cannot run query');
                    }

                    $userInfo = array();
                    $loginInUser = null;
                    while ($row = mysqli_fetch_assoc($result)) {
                        $rowColor = "class='success'";
                        if ($row["fromUserName"] == $userName) {
                            $rowColor = "class='danger'";
                        }
                        echo " <tr " . $rowColor . ">";
                        echo " <td>" . $row["transactionKey"] . " </td>";
                        echo " <td>" . $row["fromUserName"] . " </td>";
                        echo "  <td>" . $row["ToUserName"] . "</td>";
                        echo " <td>" . $row["Amount"] . "</td>";
                        echo " </tr>";
                    }
                    mysqli_free_result($result);
                }
                mysqli_close($connect);
                ?>

            </tbody>
        </table>
    </div>
</body>
<?php require 'footerTab.php' ?>
```

3. adım olarak `GET` olarak userToken parametresini alan komutlarda güncellemek:
```hmtl
<label for='ToUserName'>Alıcının Kullanıcı Adı:</label>
<input type='text' name='ToUserName' class="form-control" id='ToUserName' maxlength="50" required />
<label for='Amount'>Miktar:</label>
<input type='text' name='Amount' class="form-control" id='Amount' maxlength="50" required />
<input type="hidden" name="fromUserName" value="<?= $_GET['userToken']; ?>" />
```

4. adım olarak `PHP` kodundaki `GET` olarak userToken parametresini alan komutlarda güncellemek:
```php
$userToken = $_GET['userToken'];
```

bu adımda userToken bilgisini aldıktan sonra onu kullanarak kullanıcıya ait userName bilgisine ulaşmam gerek bu yüzden kodumuzda userToken bilgisini ve veritabanına bağlandıktan sonra bu kod parçacığı ekleyeceğiz:
```php
//get user name from token
$query = "select userName from login  where userToken='" . $userToken . "'";
$result = mysqli_query($connect, $query);
if (!$result) {
    die(' Error cannot run query');
}
$userName = null;
while ($row = mysqli_fetch_assoc($result)) {
    $userName = $row["userName"];
    break; // to be save
}
```

bu durumda giriş yapmak istediğimizde URL'deki gönderilen parametreleri bu şekilde göreceğiz

<div align="center">
    <img src="https://github.com/yasir723/solution-of-query-string-attack/assets/111686779/71eeb696-de95-4f33-b1fd-8d18bea3b766">
</div>






