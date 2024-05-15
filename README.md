# Solution of Query string attack (Sorgu dizisi saldırısı Çözümü)
Bu saldırıyı anlatım sayfasında gördüğümüz gibi hackerler olarak hassas bilgileri url ile parametre olarak gönderilen bilgileri kullanarak nasıl oturum çalabildiğimizi öğrendik ancak bu sayfada geliştirici olarak bu saldırdan geliştirdiğimiz sistemi nasıl kuruyabileceğimizi öğreneceğiz, öncellikle hiçbir durum bizi hassas bilgileri url ile göndermemize zorunda bırakamaz, zira hassa bilgileri asla url ile gönderilmemesi gerek. ileri saldırılarda hassas bilgilerde güvenli bir şekilde nasıl göndeirlmesi gerektiğini anlatılmıştır. ancak bu sayfada eğer göndermek istiyorsak bu şekilde bir yaklaşım kullanabiliriz. 

Giriş yapıp hareketler sayfasına gittiğimizde urlde userName parametresini gönderiyoruz bu aşağıdaki fotoğrafta göreceğiz:

<div align="center">
    <img src="https://github.com/yasir723/solution-of-query-string-attack/assets/111686779/8830758a-edfc-4459-89b5-119a5b5af204">
</div>

ancak userName genellikle kolay tahmin edilebilen bir terimdir bu yüzden kullanıcıya ait userToken gibi zor tahmin edilebilen bir terim parametre olarak göndermemiz gerek, ardından bu parametreyi sunucu tarafında kullanarak kullanıcının userName özelliğini ulaşırız.

UserToken: genellikle kullanıcı oturumlarını yönetmek için kullanılan bir kimlik belgesidir. Rasgele oluşturulur ve kullanıcının kimliğini doğrulamak için sunucu tarafından kullanılır.
<div align="center">
    <img src="https://github.com/yasir723/giris-dogrulamanin-atlatilmasi-ve-kisitlamalarin-asilmasi-cozumu/assets/111686779/8a90127b-3a3b-48d3-a9d0-8c793164164a">
</div>

veritabanındaki kullanıcı tablosuna bakarasak bu şekilde göreceğiz:
<div align="center">
    <img src="https://github.com/yasir723/giris-dogrulamanin-atlatilmasi-ve-kisitlamalarin-asilmasi-cozumu/assets/111686779/8a90127b-3a3b-48d3-a9d0-8c793164164a">
</div>

işte userToken gibi bilgisi zor tahmin edilen bir terim olduğu için bu şekilde kullanarabiliriz ancak güvenlik açısından en iyi yöntem değildir. kullanıcı kimliğini doğrulamak için kullandığımız sunucu tarafındaki kod (login.php sayfası) bu şekilde username parametre olarak gönderiyoruz:
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



işte userName yerine bunu göndermemiz gerek
