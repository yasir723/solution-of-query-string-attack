# Solution of Query string attack (Sorgu dizisi saldırısı Çözümü)
Bu saldırıyı [anlatım sayfası](https://github.com/yasir723/query-string-attack)nda gördüğümüz gibi, hackerlar olarak URL'deki parametreleri kullanarak hassas bilgileri ele geçirip oturum çalabildiğimizi öğrendik. Bu sayfada, geliştirici olarak bu tür saldırılardan sistemimizi nasıl koruyabileceğimizi öğreneceğiz. Öncelikle, hiçbir durumda hassas bilgileri URL ile göndermemeliyiz ki bu ciddi bir güvenlik açığı oluşturur. İleri saldırılarda, hassas bilgilerin güvenli bir şekilde nasıl gönderilmesi gerektiği anlatılmıştır. Ancak bu sayfada, eğer hassas bilgileri göndermemiz gerekirse, bunu nasıl güvenli hale getirebileceğimiz üzerinde duracağız.


Örnek olarak kullandığımız web sitede giriş yaptıktan sonra hareket sayfasına gittiğimizde URL'de `userName` biglisini parametre olarak iletildiğini farkedeceğiz:

<div align="center">
    <img src="https://github.com/yasir723/solution-of-query-string-attack/assets/111686779/8830758a-edfc-4459-89b5-119a5b5af204">
</div>

`userName` genellikle kolay tahmin edilebilen bir terimdir. Bu nedenle, kullanıcının kimliğini belirlemek için `userToken` gibi zor tahmin edilebilen bir terimi parametre olarak göndermek daha güvenlidir. Daha sonra, bu parametreyi sunucu tarafında kullanarak kullanıcının `userName` özelliğine erişebiliriz.

`UserToken:` genellikle kullanıcı oturumlarını yönetmek için kullanılan bir kimlik belgesidir. Rasgele oluşturulur ve kullanıcının kimliğini doğrulamak için sunucu tarafından kullanılır.
<div align="center">
    <img src="https://github.com/yasir723/solution-of-query-string-attack/assets/111686779/71eeb696-de95-4f33-b1fd-8d18bea3b766">
</div>

Veritabanındaki kullanıcı tablosuna baktığımızda verileri bu şekilde göreceğiz:

<div align="center">
    <img src="https://github.com/yasir723/solution-of-query-string-attack/assets/111686779/8d46b241-825c-4941-8740-3b7e674b8473">
</div>


Kullanıcı tablosuna baktığımızda, userToken gibi tahmin edilmesi zor bir terim kullanarak güvenlik önlemlerini artırabiliriz. Ancak, bu yöntem, tam anlamıyla güvenlik sağlamak için yeterli değildir.


Kullanıcı kimliğini doğrulamak için sunucu tarafındaki kod, `login.php`:
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
Bu koda dikkat ettiğimizde, `Hareketler` butonuna tıklandığında, `userName` parametresini göndererek kullanıcı kimliğini iletir. Ancak `userToken` daha zor tahmin edilmesi açıcından `userName` yerine `userToken` göndermek istiyoruz, bu nedenle `userName` olarak belirtilen her yeri `userToken` olarak değiştireceğiz.

İlk adım olarak, giriş yapan kullanıcının `userName` bilgisi yerine userToken bilgisini bir değişkene atarız

```php
$userToken=null;
while ($row= mysqli_fetch_assoc($result)) {
    $userToken= $row["userToken"];
    break; // to be save
}
```

2. adım olarak Başarılı bir girişin ardından, kullanıcının `Hareketlerim` butonuna tıkladığında yönlendirileceği linkte, `userName` yerine `userToken` parametresini kullanmamız gerekiyor. Bu nedenle, doğru bir şekilde kullanmak için kullanıcıya ait `userToken`'ı kullanacağız.

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

Tam olarak `login.php` sayfası güncel kodu bu şekilde olacaktır:
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


Kullanıcı `Hareketlerim` butonuna tıkladığında, artık `userName` yerine `userToken` bilgisi gönderilecektir. Bu bilgiyi kullanacak olan `myActivities.php` dosyasını da güncellememiz gerekiyor. Kodun güncellenmiş hali şu şekilde olmalıdır:

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

3. adım olarak, html kodundaki `userName` parametresini `GET` olarak alan komutları `userToken` olarak güncellemek:

```php
<label for='ToUserName'>Alıcının Kullanıcı Adı:</label>
<input type='text' name='ToUserName' class="form-control" id='ToUserName' maxlength="50" required />
<label for='Amount'>Miktar:</label>
<input type='text' name='Amount' class="form-control" id='Amount' maxlength="50" required />
<input type="hidden" name="fromUserName" value="<?= $_GET['userToken']; ?>" />
```

4. aadım olarak, PHP kodundaki `userName` parametresini `GET` olarak alan komutları `userToken` olarak güncellemek:
```php
$userToken = $_GET['userToken'];
```

Bu adımda, userToken bilgisini aldıktan sonra, bu bilgiyi kullanarak kullanıcıya ait userName bilgisine ulaşmamız gerekiyor. Çünkü işlemlerimizde userName bilgisine ihtiyaç duyuyoruz. Bu yüzden, userToken bilgisini aldıktan ve veritabanına bağlandıktan sonra aşağıdaki kod parçacığını eklememiz gerekiyor:

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

Son olarak, giriş yaptığımızda URL'de gönderilen parametreleri şu şekilde göreceğiz:

<div align="center">
    <img src="https://github.com/yasir723/solution-of-query-string-attack/assets/111686779/71eeb696-de95-4f33-b1fd-8d18bea3b766">
</div>



Bu şekilde hassas bilgileri sorgu olarak ileterek sistemi daha güvenli hale getirmiş olduk.


