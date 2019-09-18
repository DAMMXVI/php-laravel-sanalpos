# Teb Sanal Pos Entegrasyon Modülü PHP Laravel
Laravel tabanlı yazdığım  sanal pos kütüphanesi. 3D Secure modüllü olarak hazırlanmıştır.

## Kurulum
```
<?php

namespace App\Http\Libraries;

class XbankPayment{

	private $MerchantId, $PurchaseAmount, $InstallmentCount, $ExpiryDate, $Cavv, $Lang, $MerchantPassword;
	private $Pan, $SuccessURL, $FailureURL, $Currency, $Year, $Month, $MPIServiceUrl, $TxnType, $SecureType;

	public function postProc(Request $request)
	{
		$this->MPIServiceUrl = 'bankadan alinacak mpiserviceurl'; 
		$this->SuccessURL = '3D onayı işlemi başarılıysa dönülecek web sayfası';
		$this->FailureURL = '3D onayı işlemi başarısızsa dönülecek web sayfası';
		$this->Currency = '949'; //949 ₺ para birimini göstermektedir.
		$this->MerchantId = 'bankadan verilen şube id';
		$this->MerchantPassword = 'bankadan verilen şube şifresi';
		$this->TxnType = 'Auth';    //İşlem tipi
		$this->SecureType = '3d_pay'; //işlem tipi 3d--3d_pay--3d_hosting
		$this->Lang = 'tr';

		$this->Pan = $request->CardNumber();                //Kart numarası (**** **** **** ****)
		$this->ExpiryDate = $request->ExpriyDate();         //Kart son kullanım tarihi (ay + yıl)
		$this->Cavv = $request->cardcvc;                    //Kart cvv numarası
		$this->Year = $request->cardexpirationyear;         //Kart son kullanım tarihi (yıl)
		$this->Month = $request->cardexpirationmonth;       //Kart son kullanım tarihi (ay)
		$this->PurchaseAmount = $request->price;	        //Ödeme miktarı
		$this->InstallmentCount = $request->installment;    //Taksit sayısı
        get3D();
	}
    
	public function get3D()
	{
		$Rnd = microtime(); //Rastgele üretilen bir değer
		$TxnType = $this->TxnType;  //işlem tipi
		$Hash = $this->generateHashTest($Rnd,$TxnType); //Oluşturulan hash kodu

		$ch = curl_init();
		curl_setopt($ch,CURLOPT_URL,$this->MPIServiceUrl); //Website içeriği getirilecek URL adresi
		curl_setopt($ch,CURLOPT_POST,TRUE);		//Post onayı
		curl_setopt($ch,CURLOPT_SSL_VERIFYPEER,FALSE); //cURL’un eş sertifikasını doğrulaması için FALSE , değilse TRUE olmalıdır.
		curl_setopt($ch,CURLOPT_HTTPHEADER,array("Content-Type"=>"application/x-www-form-urlencoded")); //HTTP başlık alanlarını içeren bir dizi değerin tanımlanmasını sağlar.
		curl_setopt($ch,CURLOPT_RETURNTRANSFER,TRUE);//true olarak setlenmesi durumunda çıktılar string olarak sitede listelenecektir.
		curl_setopt($ch,CURLOPT_POSTFIELDS,"clientid=".$this->MerchantId."&storetype=".$this->SecureType."&hash=".$Hash."&islemtipi=".$TxnType."&amount=".$this->PurchaseAmount."&currency=".$this->Currency."&okUrl=".$this->SuccessURL."&failUrl=".$this->FailureURL."&lang=".$this->Lang."&rnd=".$Rnd."&pan=".$this->Pan."&Ecom_Payment_Card_ExpDate_Year=".$this->Year."&Ecom_Payment_Card_ExpDate_Month=".$this->Month."&cv2=".$this->Cavv."&taksit=".$this->InstallmentCount);
		//Bir HTTP POST işleminde gönderilecek verinin tamamını deger olarak olır. 
		$resultXml = curl_exec($ch);	//işlem isteği mpi'a gönderiliyor.
		print_r($resultXml);
		exit;		
	}	
    public function generateHashTest($rnd,$txnType) //Banka tarafından gönderilecek hash ile doğrulama yapmak için hash oluşturulması
	{ 
		$oid = $this->VerifyEnrollmentRequestId;       
		$hashstr = $this->MerchantId.$oid.$this->PurchaseAmount.$this->SuccessURL.$this->FailureURL.$txnType.$this->InstallmentCount.$rnd.$this->MerchantPassword;
        $hash = base64_encode(pack('H*',sha1($hashstr))); 
		return $hash;
	}
}
```
