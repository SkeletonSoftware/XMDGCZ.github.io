---
layout: post
title: "Publikace Xamarin.Android aplikací" 
categories:
            - "Xamarin Forms Úvod"
            - "HowTo"
author: "Vojtěch Mádr"
---

Aplikace je hotová a otestovaná, ale co musím udělat proto, aby se dostala ke koncovým zákazníkům?

<!--excerpt-->

Následující článek se dělí na několik částí:
- [Privátní/Veřejné klíče, Certifikát a Keystore](#KeyStore)
- [Podepsání aplikace manuálně](#AppSigningManual)
- [Podepsání aplikace ve Visual Studio 17](#AppSigningVS)
- [Správa podepisujících klíčů](#KeyManager)
- [Lokální export .apk](#LocalExport)

## Privátní/Veřejné klíče, Certifikát a Keystore
{: #KeyStore }
Při buildu i nasazení aplikace je potřeba k APK připojit [Public-key certifikát](https://developer.android.com/studio/publish/app-signing). Jedná se o identifikační certifikát , který obsahuje veřejnou složku z klíče (tzv. app signing key), který je tvořen veřejnou a privátní částí a metadaty popisující jeho vlastníka. Tento cerifikát následně koresponduje s privátní složkou klíče a jednoznačně určí jeho vlastníka. Tyto privátní klíče jsou uložené v binárním souboru tzv. Keystoru, který budeme používat pro podepisování aplikace.

![Cert](/assets/posts/courses/2018-12-08-PublicationAndroid/Screenshot-03.png)


V případě, že aplikaci debugujete, Xamarin.Android automaticky vyhledá Debug.Keystore a podepíše vaše APK v něm obsaženým klíčem. Tento klíč je ale určen a připraven pouze pro debugování a tudíž ho nemůžete použit pro nasazení do Google Play Storu.
Nalézá je typicky v těchto složkách :

```
* /.android/ (OS X and Linux)
* C:\Users\<user>\.android\  (Windows Vista and Windows 7, 8, and 10)
```

Pro nasazení aplikace do ostrého provozu Vám nezbyde nic jiného než aplikaci podepsat vaším certifikátem. Visual studio Vám nabízí automatickou tvorbu KeyStoru.

![KeyStoreEditor](/assets/posts/courses/2018-12-08-PublicationAndroid/Screenshot-02.png)

### Popis důležitých položek:

* Alias: Zadání aliasu (pojmenování) pro privátní klíč
* Password: Zadání hesla pro - "KeyStore" (Store password) a zároveň "Privátního Klíče (Key password)" + jeho potvrzení (Pokud toto generujete tento klíč pomocí Android Studia, můžete tyto hesla zadat zvlášť).
* Validity (roky): Doba, po kterou bude klíč platit. Klíč musí být validní alespoň 25 let. Po jeho vypršení 
* Další informace o uživateli : Tyto informace nejsou zobrazeny v aplikaci, ale budou přidány do certifikátu, který je součastí APK.


nebo [manuálně](https://docs.microsoft.com/en-us/xamarin/android/deploy-test/signing/manually-signing-the-apk#Sign_the_APK_with_jarsigner).


Pokud se chcete podívat na údaje (nejčasteji Fingerprinty) v privátním klíči, zavítejte [zde](https://docs.microsoft.com/en-us/xamarin/android/deploy-test/signing/keystore-signature?tabs=macos)!

## Podepsání aplikace manuálně
{: #AppSigningManual}

Podepsání aplikace probíhá ve dvou krocích : Zipalignování APK a podepsání pomocí apksigner
(apksigner je dostupný od Android SDK v24.0.3)


1.Zipalign the APK 

Zipalign je optimalizační proces, který zrychluje práci s aplikací. Xamarin.Android v runtimu hlídá jestli je na APK aplikovaný Zipalign a pokud není, aplikaci systém nedovolí spustit.
```
zipalign -f -v 4 mono.samples.helloworld-unsigned.apk helloworld.apk
```

2.Sign the APK

Aplikace poté snadno podepíšete pomocí appsigneru.
```
apksigner sign --ks xample.keystore --ks-key-alias publishingdoc mono.samples.helloworld.apk
```

Tento proces za Vás udělá Xamarin Studio při buildu a také při nasazení aplikace.

## Podepsání aplikace ve Visual Studio 17
{: #AppSigningVS}

Aplikaci je možné podepsat existujícím keystore přímo z IDE Visual Studio 17. K této možnosti se lze dostat pomocí vlastností .Android projektu v záložce Android Package Signing.

![VSKeyStore](/assets/posts/courses/2018-12-08-PublicationAndroid/VSKeyStore.png)


## Správa podepisujících klíčů
{: #KeyManager }

V současné době máte dvě možnosti, jak uchovat Klíč pro podepsání aplikace. Tyto možnosti si popíšeme v následujících řádcích:

### Použití App Signing na Google Play (Doporučený způsob)

Pokud použijete na správu klíčů Google Play, budete muset mít k dispozici dva klíče - zmiňovaný "App Signing Key" a tzv. "Upload Key".

1. App Signing klíč bude napevno uložen na Google Play a tudíž nemůže dojít k jeho ztrátě. 

2. Upload klíčem budete aplikaci podepisovat při každém nasazení. V případě jeho ztráty ale není problém vygenetovat nový klíč. Stačí informovat Google Play podporu, která Vám jeho výměnu umožní.

![GooglePlayKey](/assets/posts/courses/2018-12-08-PublicationAndroid/Screenshot-04.png)


### Použití vaší interní správy klíčů

Pokud si myslíte, že jste schopný si svůj KeyStore (a v něm uchovaný App Signing Key) pohlídat, není potřeba ho přidávat do Google Play. Musíme mít ale na paměti, že v případě jeho ztráty (či ztráty jeho hesla) musíme aplikaci nasadit pod novou registrací na Google Play.

## Lokální export .apk
{: #LocalExport}

Pokud je potřeba exportovat aplikaci pouze lokálně např. pro interní distribuci mezi testery, Tak je možné vytvořit achiv (.apk) přímo v IDE.

Pro VisualStudio 17 stačí kliknout na Android projekt pravým tlačítkem a zvolit možnost Archivovat. Pro zajištění funkčnosti nesmí být povolen ``Fast Deployment``, který je zapnutý defaultně v Debug konfiguraci buildu. Nejjednoduší je přepnout Build konfiguraci na Release a poté toto sestavení archivovat.

![GooglePlayKey](/assets/posts/courses/2018-12-08-PublicationAndroid/VS-AndroidArchive.png)

Poté stačí kliknout na tlačítko Distribute, zadat keystore a vybrat místo pro uložení do Pc.

> Před archivací je dobré nejdříve nové nastavení sestavení vyzkoušet (alespoň build).

### Přidání další architektury CPU do Release
 
Defaultně Release konfigurace podporuje pouze ARM architekturu CPU. Pokud je při spouštění na emulátoru (většinou x86) zobrazena chyba ``Xamarin package does not support for CPU Architecture ...", nebo ``Deployment failed. Architecture not supported.`` Tak stačí ve vlastnostech Android projektu -> Android Options -> Advanced, přidat další architekturu v rolovací liště Supported architectures.

![GooglePlayKey](/assets/posts/courses/2018-12-08-PublicationAndroid/VSAddCPUArchitecture.png)

### Zdroje:

[Manage your app signing keys](https://support.google.com/googleplay/android-developer/answer/7384423?hl=en)

[Manually Signing the APK](https://docs.microsoft.com/en-us/xamarin/android/deploy-test/signing/manually-signing-the-apk#Sign_the_APK_with_jarsigner)

[Publishing Google Play](https://docs.microsoft.com/en-us/xamarin/android/deploy-test/publishing/publishing-to-google-play/?tabs=macos)

[CPU Architectures](https://docs.microsoft.com/cs-cz/xamarin/android/app-fundamentals/cpu-architectures?tabs=windows)