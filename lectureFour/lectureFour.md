# Dördüncü Ders - Daha Üst Düzey Akıllı Sözleşmeler

## İçindekiler
* Programlamada `Daha Üst Düzey` Metodolojisinin Anlaşılması
* Agoric'in Bu `Daha Üst Düzey` Metodolojiyi Nasıl Uyguladığı
    * Türetilmiş erights
    * Davetiyeler
    * Yeniden Kullanılabilir Akıllı Sözleşmeler

## Programlamada `Daha Üst Düzey` Metodolojisinin Anlaşılması
_`Daha Üst Düzey` Programlama Metodolojisi_'nin en somut uygulaması daha üst düzey fonksiyonlardır (HOF).
[Definition](https://en.wikipedia.org/wiki/Higher-order_function) tanımına göre bir daha üst düzey fonksiyon en azından
aşağıdakilerden birini yapar:

* Bir veya daha fazla fonksiyonu argüman olarak alır
* Sonuç olarak bir fonksiyon döndürür

Yukarıdaki tanım genel hesaplama için geçerlidir, bu nedenle hem bilgisayar bilimlerini hem de matematiği kapsar.

JavaScript daha üst düzey fonksiyonları yerleşik olarak destekler. Aslında daha üst düzey
fonksiyonları kullanan bazı yerleşik metotlara sahiptir. İşte birkaç örnek;

* Array.prototype.map()
* Array.prototype.filter()
* Array.prototype.reduce()
* ...

Ve liste devam eder. `Array.prototype`, programcıların diziler üzerinde yineleme gerektiren birçok işlemi olabileceği için bu daha üst düzey metodlara sahiptir. Bu metodların mantığını iyi örneklerle anlamak için [bu makaleyi](https://medium.com/javascript-scene/higher-order-functions-composing-software-5365cf2cbe99)
kontrol ettiğinizden emin olun.

## Agoric'in Bu `Daha Üst Düzey` Metodolojiyi Nasıl Uyguladığı
Agoric, bu metodolojiyi uygulamak için somut daha üst düzey fonksiyonların üzerinde birkaç soyutlama kullanır. Biz
daha üst düzey zihniyeti gözlemleyebileceğimiz ve aynı zamanda bu üç örneğin nasıl sağlam bir platform oluşturmak için birlikte çalıştığı üç örneğe odaklanacağız.

* Türetilmiş erights
* Davetiyeler
* Yeniden Kullanılabilir Akıllı Sözleşmeler

Haydi onları tek tek inceleyelim.


### Türetilmiş erights
`Türetilmiş eright`ın ne olduğunu ve nasıl kullanabileceğimizi anlamak için, [İkinci Dersi](../lectureTwo/lectureTwo.md) hatırlamalıyız:

"*...Sözleşmeler, bu korunan alanlar arasında hakların alışverişini sağlar.*"[2]

<img src="./images/contractHost.png" width='60%'> 

*Şekil 4: Örnek Takas Diyagramı[3] (İkinci dersten)*

İkinci derste, sözleşmelerin karşılıklı olarak şüpheli taraflar arasında hakların ticaretini nasıl mümkün kıldığını ve bu sistemin işlemesi için neden bir `Sözleşme Ev Sahibi`'ne ihtiyaç duyulduğunu anlatmıştık. Alice'nin bir süreliğine bir varlığı tutması için Bob'a ödeme yaptığını ve süre dolmadan önce bu varlığı 10 dolar karşılığında satın almak isteyebileceğini düşünelim. Bu bir opsiyon hakkı değil mi? Ve Alice ve Bob bu sözleşmeyi başlattıktan sonra, varlığın spot fiyatı yükselmeye başlar. Şimdi Alice, Fred'in sahip olduğu başka bir varlık istiyor, bu yüzden Fred'e opsiyon hakkını Fred'in sahip olduğu varlıkla takas etmeyi teklif ediyor. 10 dolar karşılığında varlığı satın alma hakkı kendisi bir eright'tir ve Alice bu eright'ı başka bir varlıkla ticaret yapmak için kullanıyor.

Bu bizi aşağıdaki duruma getirir;

<img src="./images/layeredGames.png" width='60%'>

Elektronik bir hakın bir eright olarak sınıflandırılabilmesi için hangi özelliklere sahip olması gerektiğini hatırlıyor musunuz?
* Özel
* Analiz Edilebilir

Yukarıdaki her iki özelliği de elde etmek için güvenilir bir üçüncü tarafın tanıtılması gerekiyor. ERTP'de bu tarafa `Issuer` (Düzenleyici) diyoruz. Bu bizi şu soruya getirir: Alice'in opsiyon hakkını kullanma hakkının düzenleyicisi kimdir?

### Davet
Davetler, aslında `Payment`'lerdir (Ödemelerdir). Bu tasarım seçimini anlamaya çalışalım;
* Davetlerin düzenleyicisi kimdir?
    * Tek bir düzenleyicisi mi var? Yoksa her sözleşmenin kendi davet düzenleyicisi mi var?
* Davetlerin `AssetKind`'ı nedir?
* Davetleri ERTP varlıkları olarak oluşturduğumuzda hangi tür faydalar elde ediyoruz?

#### Davetlerin düzenleyicisi kimdir?
Davetlerin nasıl oluşturulduğunu hatırlıyor musunuz? Davetler, aşağıdaki yöntemi kullanarak `Zoe Contract Facet` ile oluşturulur;

```js
const invitation = zcf.makeInvitation(offerHandler, 'Description', customProperties = undefined, proposalShape = undefined);
```

Bu, davetlerin yalnızca bir sözleşmeden oluşturulabileceği anlamına gelir. [Agoric Docs](https://docs.agoric.com/reference/zoe-api/zoe.html#e-zoe-getinvitationissuer) üzerinde aşağıdaki gibi yararlı bir örnek vardır:

```js
const invitationIssuer = await E(Zoe).getInvitationIssuer();
// Burada bir kullanıcı olan Bob, Alice'den güvenilmeyen bir davet aldı.
// Bob, Zoe'den alınan güvenilir **InvitationIssuer**'i kullanarak
// güvenilmeyen daveti güvenilir bir davete dönüştürür
const trustedInvitation = await invitationIssuer.claim(untrustedInvitation);
const { value: invitationValue } =
    await E(invitationIssuer).getAmountOf(trustedInvitation);
```

Yukarıdaki kod parçacığı, Alice'in güvenilmeyen bir davete nasıl yaklaştığını gösterir. İzlediği adımlar;
1. Davetin güven kaynağını alır
2. Daveti, ERTP'nin `issuer.claim()` yöntemi kullanılarak doğrular
3. Doğrulanmış davetin içindeki verileri kontrol eder

Birinci adım, "_Davetlerin düzenleyicisi kimdir?_" sorumuzun açıkça cevabını verir, `invitationIssuer`'ı Zoe'den alırız. Herhangi bir örnek, kurulum veya başka türden argümanları belirtmeden. Dolayısıyla, tüm davetlerin düzenleyicisinin Zoe (Sözleşme Ev Sahibi) olduğunu güvenle söyleyebiliriz.

#### Davetlerin `AssetKind`'ı nedir?
Bu soru, yukarıdaki bölümde zaten yanıtlandı ancak dolaylı bir şekilde. `zcf.makeInvitation()`'ın üçüncü argümanına dikkat edin:

```js
const invitation = zcf.makeInvitation(offerHandler, 'Description', customProperties = undefined, proposalShape = undefined);
```

`customProperties` argümanı, davetin değerine giden bir `Nesne`dir, böylece müşteriler, Alice gibi, verilerini kullanarak davete güven oluşturabilir. Bu nedenle, davetlerimizin `non-fungible` (birbirinin yerine geçemeyen) olduğunu ve bu durumda varlık türünün `COPY_SET` olduğunu sonuçlayabiliriz.

#### Davetleri ERTP varlıkları olarak oluşturduğumuzda hangi tür faydalar elde ediyoruz?

[Türetilmiş erights]() bölümündeki örneğimize geri dönelim. Alice, bir davet yerine bir kullanıcıKoltuğu sunsaydı ne olurdu? Fred, kullanıcıKoltuğunun gerçekten ona kapalı çağrı opsiyonunu döndüreceğine güvenebilir miydi? Maalesef Fred, Alice'in kapalı çağrı opsiyonunu kullanma hakkını döndürecek bir koltuk olduğunu iddia ettiği nesnenin gerçekten Alice'in söylediği şey olduğundan ve keyfi bir nesne olmadığından emin olmanın bir yoluna sahip değil. Öte yandan, eğer Fred bu koltuğu bir daveti kullanarak alırsa, aşağıdaki garantilere sahip olur;

* Davet, bu sözleşme ile ilgilidir (aksi takdirde sözleşme karşılığında bir koltuk vermez)
* Sözleşmenin doğrudan verdiği koltuk, sözleşmeye göre çalışandır

Fred'in yukarıdaki garantilere sahip olabilmesinin nedeni, Agoric belgelerindeki [bu bölümde](https://docs.agoric.com/reference/zoe-api/zoe.html#e-zoe-getinvitationdetails-invitation) açıklanmıştır ve aşağıdaki kod parçacığı ile gösterilmiştir:

```js
const invitation = await invitationIssuer.claim(untrustedInvitation);
const invitationValue = await E(Zoe).getInvitationDetails(invitation);
```

Bir davet, non-fungible bir ERTP varlığı olduğundan, değerlerinin `Object` türünde olduğunu biliyoruz. Zoe, `await E(Zoe).getInvitationDetails(invitation)` kullanarak belirli bir davetin değerini rahat bir şekilde okumamıza izin verir. Bu işlemin 

```js
const { value: invitationValue } = await E(invitationIssuer).getAmountOf(trustedInvitation);
```

yukarıdaki metotla eşdeğer olduğunu unutmayın, çünkü Zoe zaten `invitationIssuer`ı bilir.

Bir davetin değerinin içeriği aşağıdaki gibidir:

```js
const invitationValue = {
  installation, // Sözleşmenin Zoe kurulumu.
  instance, // Bu davetin için olduğu sözleşme örneği.
  invitationHandle, // Bu Davete atıfta bulunmak için kullanılan bir Handle.
  description, // Bu Davetin amacını açıklar. Daveti, sözleşmedeki rolü ile eşleştirmek için kullanın.
  ?customProperties, // Geliştirici, müşterilerin güven sağlamak için daha fazla inceleme yapmaları için bunu geçer
}
```

Fred, yukarıdaki bilgileri kullanarak güvendiği ve iş yapmayı istediği verilerle karşılaştırabilir.

Bir davetin, basit bir nesne kullanmanın üzerinde bir garanti daha sağladığı var. Bunun ne olduğunu tahmin edebilir misiniz?

* İpucu: **Davetler ödemelerdir.**

Tamam, `Issuer` sayesinde doğrulama özelliğini başardık. Peki ya tekil özellik? Teklifler uygulandığında davetler yakılır. Böylece bir davetin yalnızca BİR KEZ kullanıldığından emin olabiliriz. Yani Fred, bu hakkı yalnızca kendisinin kullandığını bilebilir.

#### Davetler, `Üst-Düzey Kompozisyon`u nasıl sağlar?
Şimdiye kadar davetlerin nasıl çalıştığı ve ne oldukları hakkında konuştuk. Harika, ama üst düzey kompozisyon bakış açısından bu nasıl yardımcı olur?

1. İlk olarak, bir davet oluşturma şeklimizi gözlemleyelim:
  ```js
  const invitation = zcf.makeInvitation(offerHandler, 'Description', customProperties = undefined, proposalShape = undefined);
  ```

`offerHandler`ın aslında bir fonksiyon olduğunu fark edin. Yani davetler, `Üst-Düzey Fonksiyonları` kullanır.

2. Türetilmiş erights'ın güvenli bir şekilde ticaretini yapmamıza yardımcı olurlar ki bu, temel eright'ın üst düzey bir versiyonudur.


### Reusable Smart Contracts
Let's see some code now.
* [Funded Call Spread](https://github.com/Agoric/agoric-sdk/blob/master/packages/zoe/test/unitTests/contracts/test-callSpread.js)


## Recommended Papers
* [Reasoning about Risk and Trust in an Open Word](https://agoric.com/assets/pdf/papers/reasoning-about-risk-and-trust-in-an-open-world.pdf) <br>
* [Distributed Electronic Rights in JavaScript](https://agoric.com/assets/pdf/papers/distributed-electronic-rights-in-javascript.pdf) <br>
* [The digital path](https://agoric.com/assets/pdf/papers/digital-path.pdf)<br>

## Resources
[1] https://www.youtube.com/watch?v=iyuo0ymTt4g <br>
[2] https://www.codecademy.com/learn/game-dev-learn-javascript-higher-order-functions-and-iterators/modules/game-dev-learn-javascript-iterators/cheatsheet
<br>
[3] https://github.com/Agoric/agoric-sdk/blob/master/packages/ERTP/NEWS.md <br>
[4] https://github.com/Agoric/documentation/issues/360 <br>
[5] https://github.com/Agoric/agoric-sdk/issues/81 <br>
[6] https://agoric.com/wp-content/uploads/2022/08/Agoric-White-Paper-v1.1.pdf <br>
[7] https://docs.agoric.com/guides/js-programming/#vats-the-unit-of-synchrony <br>
[8] https://github.com/Agoric/agoric-sdk/blob/master/packages/zoe/docs/zoe-zcf.md <br>
[9] https://docs.agoric.com/reference/zoe-api/zoe.html#e-zoe-startinstance-installation-issuerkeywordrecord-terms-privateargs <br>
