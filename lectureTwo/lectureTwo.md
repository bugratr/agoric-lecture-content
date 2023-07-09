# İkinci Ders - ERTP ve Zoe'ya Giriş
## İçindekiler
* ### ERTP
  * Elektronik hak nedir?
  * Elektronik hak vs eright
  * ERTP nedir?
  * Örnek `eright`ların uygulanması
  * Agoric'teki ERTP

* ### Zoe
  * Bağlam
  * Bir Zoe Sözleşmesinin Yapısı
  * Teklifler
  * Zoe'nin İki Yüzü
  * Sözleşme Geliştirme Deneyimi

## ERTP

### Elektronik hak nedir?
* #### Nesne Yetenekleri
  `Elektronik hak`ın ne olduğunu anlamak için öncelikle `Nesne Yetenekleri` adı verilen konsepta bir göz atmamız gerekiyor.

  ![Şekil 1: Granovetter Diyagramı](../literatureWork/images/granovetterOperator.png)

  Yukarıdaki diyagram, insanlar arasındaki sosyal ilişkileri gösterir ve sosyolog Mark Granovetter tarafından tasarlanmıştır. Nesne Yetenekleri disiplini, bu diyagrama `Granovetter Operatörü` adını verir ve nesneler arasındaki ilişkileri göstermek için kullanır.

  Temelinde, bir `ocaps` (Nesne Yetenekleri) sistemi, kullanıcıları sahip oldukları nesne referanslarına göre yetkilendirir. Bir `ocaps` sisteminde, Erişim Kontrol Listeleri(ACLs) yoktur. Basitçe, yetenekleriniz sahip olduğunuz nesne referansları tarafından tanımlanır. 

* #### 'Haklar' Üzerine Bir Söz

  "Haklar, insanların planları koordine etmelerine ve kaynakların kullanımı üzerinde çıkan çatışmaları çözmelerine yardımcı olur.
Haklar, ayrı ayrı formüle edilen planlar arasında müdahaleyi önlemek için eylemlerin alanını böler, böylece karşılıklı şüphe ve rekabet eden hedeflere rağmen işbirlikçi ilişkileri mümkün kılar [15]. Bu hak temelli perspektif, dağıtılmış hesaplama sistemlerinin güvenliğini sağlama sorununa ışık tutabilir"[2]

* #### Nesnelerden Elektronik Haklara
  Nesne referansları, belirli, atanmış bir kaynak üzerinde bir dizi işlem yapma hakkını temsil eder.
Bu bağlamda, şu anda `normal` olarak bildiğimiz haklarla çözülmekte olan gerçek dünya problemlerini adreslemek için elektronik hakları kullanabiliriz. Sözleşmeler, hukuk, mahkemeler vb. kullanarak.

### Elektronik hak vs eright
`Granovetter Operatörü`nden Alice, Bob ve Carol'un bir `ocap` ortamında yaşayan bazı nesneler olduğunu düşünelim.
- Alice'nin Carol'dan bir kaynağı çağırma `hakkı` vardır.
- Alice Bob'u tanıdığı için Carol'u çağırma hakkını Bob'a geçirebilir. Alice bu hakkı ona geçirmeyi reddederse, Bob'un Carol'u çağırma yolu yoktur.
- Carol'un ne Alice'den ne de Bob'dan herhangi bir kaynağı çağırma yetkisi yoktur.

Yukarıda incelediğimiz `hak` türünün aşağıdaki özelliklere sahip olduğunu görebiliriz;
* Paylaşılabilir
* Uygulanabilir
* Şeffaf Olmayan
* Belirli

İşte Mark Miller'ın elektronik haklar ve erights arasındaki farklar hakkında [erights.org](http://erights.org/smart-contracts/index.html#ERTP) üzerinde söyledikleri;

"_Yetenekler, bir tür elektronik haklardır, ancak ticarete konu olabilen elektronik haklar için 2 önemli özellikten yoksundurlar:_

* _Münhasır haklar transferi._
* _Deneyebilirlik (bu sayede bir 3. tarafın bir takasın karşılıklı kabul edilebilir olup olmadığını belirleyebilmesi)._

_Dağıtılmış yeteneklerin üzerindeki bir sonraki katman olan ERTP, bu özellikleri sağlar. 
ERTP protokolü, fungible & non-fungible hakları, münhasır & münhasır olmayan hakları, 
ve kör veya kör olmayan transferi barındırır. **ERTP aracılığıyla manipüle edilebilen yalnızca elektronik haklara erights denir.**_"

Yukarıda incelediğimiz hakları Mark Miller'ın `eright` tanımıyla karşılaştırdığımızda, hakların
`Granovetter Operatörü`nden olan transferinin, bir elektronik hakkın `eright` olarak adlandırılması için iki ana özelliği eksik olduğunu görebiliriz. Neden?

1. Bir nesne referansı paylaşılabilir ama bir eright münhasır olmalı. Ne demek istiyoruz? <br>
   Eğer Alice, yeteneğini Bob'a geçirdikten sonra bırakırsa, Bob
   Carol'a münhasır erişime sahip olur, ancak bu bir __*münhasır*__ hak değildir çünkü Bob
   onun tek sahibi olduğunu bilemez.[1]
2. Bir yetenek şeffaf olmayan bir durumdadır, ama bir eright deneyebilir olmalıdır. Anlamı; <br>
   Deneyebilirlik, ticaret için gereklidir, çünkü satın almayı düşünmeden önce ne elde edeceğinizi belirlemeniz gerekir. Ancak, münhasır haklar yalnızca onlara münhasır eriş
* ##### How to create fungible tokens with ERTP?
  Initially;
  ```js
  const {
     issuer: quatloosIssuer,
     mint: quatloosMint,
     brand: quatloosBrand,
  } = makeIssuerKit('quatloos');  
  ```
  We call `makeIssuerKit` in order to get a fresh set of `Issuer`, `Mint` and `Brand` objects. Then we've to creat
the amount to be minted;
  ```js
  const quatloosSeven = AmountMath.make(quatloosBrand, 7n);  
  ```
  Once we have the amount, we can actually print the money using the `Mint` object,
  ```js
  const quatloosPayment = quatloosMint.mintPayment(quatloosSeven);
  ```
  A successful mint results in a `Payment` object. Payments are the money `on-the-go`, we should put it in a `Purse`
if we want to park our money. To put a payment inside a purse;
  ```js
  const quatloosPurse = quatloosIssuer.makeEmptyPurse();
  quatloosPurse.deposit(quatloosPayment);
  ```
##### ERTP ile nasıl non-fungible tokenlar oluşturulur?
İlk adımımız, oluşturduğumuz bir fungible varlık ile çok benzer fakat tek bir farkla;
```js
const {
  mint: popMint,
  brand: popBrand,
  issuer: popIssuer,
 } = makeIssuerKit('POP', AssetKind.COPY_SET);
```
`AssetKind.COPY_SET` argümanı, non-fungible tokenlar oluşturmak istediğimizde belirtilir. Fungible tokenlar için argüman `AssetKind.NAT`tir ancak bunu belirtmedik çünkü varsayılan seçenektir. Diğer olası AssetKind seçenekleri için belgelere bakabilirsiniz [buradan](https://docs.agoric.com/reference/ertp-api/ertp-data-types.html#assetkind). EmitterKit'imiz olduğunda bir sonraki adıma geçebiliriz, bu da basılacak miktarı oluşturmaktır;
```js
const popAmount = AmountMath.make(popBrand, harden([{
      organization: 'Chainboard Academy',
      courseName: 'Agoric Bootcamp',
      studentId: '12',
    }]));
```
Bir NFT'nin verisi bir dizi etrafına sarılır. Bu, non-fungible miktarları eklemek/çıkarmak istediğimizde özellikle yararlıdır. Artık basma aşamasına geçebiliriz;
```js
const popPayment = popMint.mintPayment(popAmount);  
```
NFT'leri, fungible tokenları depoladığımız gibi depolayabiliriz;
```js
const popPurse = popIssuer.makeEmptyPurse();
popPurse.deposit(popPayment);
```

## Zoe
### Context
"*...Sözleşmeler, bu korunan alanlar arasında hakların değişimini sağlar.*"[2]

<img src="./images/contractHost.png" width='60%'> 

*Figure 4: Sample Exchange Diagram[3]*

Bir kez `takas edilebilir elektronik haklar` elde ettiğimizde, bu hakların ticaretini sağlamak için güvenli bir katmana ihtiyaç duyarız.
İşte tam olarak bir sözleşmenin yaptığı şey yukarıdaki alıntıya göre. Ancak bu başka bir sorunu gündeme getirir. Sözleşmeyi kötü niyetli kullanıcılardan nasıl koruyacağız? Yukarıdaki diyagramda Alice ve Bob[2] üzerinde anlaştılar;
* Her iki hakın da ihraççıları.
* Sözleşmenin kaynak kodu.
* Sözleşmenin hangi tarafını oynayacağı.
* Ne olursa olsun, onların kabul ettiği kodu dürüstçe çalıştıracakları bir üçüncü taraf.

Bu üçüncü taraf, Alice ve Bob'ın kodlarını çalıştırmaları için karşılıklı olarak güvendikleri kişi **Sözleşme Ev Sahibi**dır. `Zoe`, Agoric tarafından Agoric ekosisteminde sözleşme ev sahibi olarak hizmet etmek üzere tasarlanmıştır. Bu, tüm akıllı sözleşmelerin kurulduğu ve çalıştığı katmandır.

`Zoe`, kullanıcılar ve geliştiriciler arasında güven oluşturur. Nasıl? Diğer büyük ağlarda (örneğin Ethereum'da) bir akıllı sözleşme geliştiricisi, sözleşme kodu içindeki kullanıcı varlıklarına tam erişime sahiptir. Bu, bazı kötü niyetli geliştiricilerin bazı kötü eylemler gerçekleştirmesine olanak sağlar. Ancak bir `ocaps` sisteminde bu asla olmamalıdır. `Ocaps` sistemindeki slogan: *Güvenlik getirme, güvensizliği kaldır.* `Zoe`, bu zihniyeti akıllı sözleşme geliştirme sürecine getirir. `Zoe` bunu `escrowing` yoluyla yapar. Bir akıllı sözleşme, karşılıklı şüpheli tarafların haklarını ticarete çeviren yerdir. `Zoe`, sözleşmedeki tüm tarafların haklarını, sözleşmede belirli bir koşul yerine gelene kadar kilitler/escrows. Bir geliştiricinin haklara doğrudan erişimi yoktur, bunun yerine gerekli mantığı `Amount`ları kullanarak uygularlar. `ERTP` bölümünden `Amount`ları hatırlıyor musunuz? İşte `ERTP` ve `Zoe` nasıl çalışır ve `erights`ın güvenli ticaretini nasıl sağlar.

### Bir Zoe Sözleşmesinin Yapısı
`Zoe`, kullanıcıları kötü niyetli geliştiricilerden koruyan *Sözleşme Ev Sahibi* olarak hareket eder, ancak aynı zamanda akıllı sözleşme geliştiricileri için yeteneklerini ve yaratıcılıklarını gösterebilecekleri zengin bir çerçevedir.

Agoric'teki akıllı sözleşmeler `Zoe` aracılığıyla dağıtılır ve erişilir. Ama, `Zoe` her yüklenen kodu akıllı sözleşme olarak kabul eder mi? Elbette hayır, akıllı sözleşmelerin aşağıdaki yapısı olmalıdır;

<details>
  <summary> 
    Örnek Sözleşme
  </summary>

```js
// @ts-check
// JSDoc yorumlarında tanımlanan türleri kontrol eder

// İçe aktarmalarınızı buraya ekleyin

// Opsiyonel: Zoe yardımcılarını kullanmayı tercih edebilirsiniz
// @agoric/zoe/src/contractSupport/index.js
import { swap as _ } from '@agoric/zoe/src/contractSupport/index.js';

// Zoe türlerini içe aktar
import '@agoric/zoe/exported.js';

/**
* [Sözleşme Açıklaması Buraya]
*
* @type {ContractStartFn}
  */
const start = (zcf, _privateArgs) => {
// ZCF: Zoe Contract Facet

// privateArgs: sözleşme sahibi tarafından sözleşme 
// koduna kamuflajlı terimlerde bulunmaması gereken sözleşmeye 
// açık olması gereken herhangi bir argüman.

// Tekliflerin işlenmesi ve davetiyelerin oluşturulması dahil 
// olmak üzere sözleşme mantığınızı buraya ekleyin.

// Örnek: Bu, otomatik olarak bir iade ödemesi veren bir 
// teklif işleyicisinin bir örneğidir.
const myOfferHandler = zcfSeat => {
zcfSeat.exit();
const offerResult = 'başarı';
return offerResult;
};

// Örnek: Bu, bir teklif yapmak için kullanılırsa, 
// `myOfferHandler`ı tetikleyen ve otomatik olarak bir 
// geri ödeme sağlayan bir davetiyedir.
const invitation = zcf.makeInvitation(myOfferHandler, 'myInvitation');

// Opsiyonel: Bu nesneye eklenen metodlar, örneğin 
// oluşturucuya kullanılabilir.
const creatorFacet = {};

// Opsiyonel: Bu nesneye eklenen metodlar, sözleşme 
// örneğini bilen herkese kullanılabilir.
// Fiyat sorguları ve diğer bilgi talepleri burada olabilir.
const publicFacet = {};

return harden({
   creatorInvitation: invitation, // opsiyonel
   creatorFacet, // opsiyonel
   publicFacet, // opsiyonel
  });
};

harden(start);
export { start };
```
</details>

Yukarıda, en basit haliyle bir `Zoe` sözleşmesi örneği bulunmaktadır. Ancak, hala bir `Zoe` sözleşmesinin tüm yapısal parçalarını içermektedir. Bu bileşenleri bir bir inceleyelim;
1. Her `Zoe` sözleşmesi, genellikle sözleşmenin son satırı olan `start` adında bir metot dışa aktarmalıdır.
   ```js
   export { start }; 
   ```
2. `start` metodu ilk argüman olarak bir `zcf` nesnesini kabul etmelidir. `zcf` `Zoe Contract Facet` anlamına gelir ve bu, akıllı sözleşme geliştiricilerin `Zoe` ile etkileşime geçmesini sağlar.
3. `function` anahtar kelimesi yerine ok işlevi tanımlamalarını kullanın;
   ```js
   // Yap
   const start = (zcf) => {
    // Metot gövdesi
   }
  
   // Yapma
   function start(zcf) {
    // Metot gövdesi
   }
   ```
4. Geleneksel olarak, çoğu `Zoe` sözleşmesi iki API döndürür: `creatorFacet` ve `publicFacet`;
   * **creatorFacet**: `creator` kelimesi, bu sözleşmeyi dağıtan kullanıcıyı ifade eder. Bu nedenle, yalnızca bu API'nin yönetim yetkisine sahip metotları içermesi gerekir.
     `creatorFacet` yalnızca sözleşmenin dağıtılması sırasında kullanılabilir. Bu yüzden yaratıcı bu referansı elinde tutmalıdır. Çünkü bir kere gitti mi, gitti.
   * **publicFacet**: Bu, sözleşmenin tüm dünyaya açtığı API'dir. `publicFacet`, `Zoe` arayüzü üzerinden erişilebilir.

### Teklifler
Tarafların güvendiği bir sözleşme ev sahibi üzerinden hakları ticaret yapmayı kabul etmeleri durumunda, sözleşmeye girmek için `Teklif` kullanılır. Bir `Teklif`in özel bir yapısı vardır ki bu, `Zoe`nun her kullanıcının haklarını güvence altına almasını sağlar. Bu güvenlik modeline ayrıca `Teklif Güvenliği` de denir ama birazdan ona geleceğiz. Şimdilik `Teklif`lere odaklanalım. Aşağıda bir `Zoe Teklifi` örneği bulunmaktadır:
```js
const userSeat = await E(zoe).offer(
  invitation,
  proposal,
  payments,
);
```

Öncelikle `offer()` metodunun argümanlarını inceleyelim:
* **invitation**: `Zoe`, kullanıcılardan sözleşmeyle etkileşim kurmak için kullanmak istedikleri metoda işaret eden bir referans bekler. 
Bu tür bir referansa `davetiye` denir. `zcf.makeInvitation()` özel bir `Zoe` metodudur ve davetiyeleri oluşturmak için kullanılır.
* **proposal**: Proposal, `give`, `want` ve `exit` olmak üzere üç önemli özelliğe sahip bir js kaydıdır. Bir örnek proposal'a göz atalım;
  ```js
  const myProposal = harden({
    give: { Asset: AmountMath.make(quatloosBrand, 4n) },
    want: { Price: AmountMath.make(moolaBrand, 15n) },
    exit: { afterDeadline

: {
      timer,
      deadline: 100n,
    }}
  });
  ```
  * `give`: Kullanıcılar, gerçek para olmamakla birlikte, ödemeye istekli oldukları tutarı belirtir.
  * `want`: Kullanıcılar, yine gerçek para olmamakla birlikte, karşılığında ne istediklerini belirtir. 
  * `exit`: Bu teklifin süresinin dolmasına kadar geçecek süre.
 
  `Asset` ve `Price`, hakları anlamlı bir şekilde tanımlamak için kullanılan anahtar kelimelerdir.  
  * **payments**: Payment, `Zoe`nun gerçek varlıkları emanete almak için kullandığı js kaydıdır. Aşağıdaki yapıya sahiptir;
    ```js
    const paymentKeywordRecord = {
        'Asset' : quatloosPayment,
    };  
    ```
    `Asset` anahtar kelimesinin, teklifin `give` özelliğindekiyle eşleştiğini fark edin. `quatloosPayment`, teklif kaydının `give` özelliğindeki birine eşit veya daha az quatloos miktarı içeren bir ERTP `Payment` nesnesidir.

    > **Önemli**: `give` bölümünde belirtilen bir miktar varsa, bu miktar için `give`
    > bölümünde karşılık gelen bir ödeme kaydı olmalıdır.
 
#### Bir Teklif Gerçekleştirildikten Sonra Neler Olur?
Bir takas, her iki yönlü bir işlem olduğundan, kullanıcılar muhtemelen ödediklerinin karşılığında bir şeyler almak isteyecektir.
```js
const userSeat = await E(zoe).offer(
  invitation,
  proposal,
  payments,
);
```

`offer()` metodu bir `userSeat` döndürür. Bir teklifin doğru bir şekilde gerçekleştirildiğinden emin olmak için, `getOfferResult` metodunu çağırmak gereklidir.
```js
const offerResult = await E(userSeat).getOfferResult()
```

`offerResult`, sözleşmenin döndürdüğü herhangi bir şeydir ve belirli bir yapısı yoktur. Sözleşme kodunun çalıştırılması sırasında bir hata olup olmadığını kontrol etmek için bunu çağırırız.

Kodun herhangi bir hata olmadan çalıştığından emin olduğumuzda, varlıklarımızı çekebiliriz;
```js
const moolaPayment = await E(userSeat).getPayout('Price');
```

İşte `Zoe` ve `Teklifler` kullanarak bir sözleşmeden `haklarımızı` nasıl çekeriz. 

### Zoe'nun İki Yüzü
*Figure 4*'te, `Sözleşme Ev Sahibi`nin içinde iki sandalye olduğunu fark edin. `Zoe`nun terminolojisi şöyledir: Her işlem bir tür `Teklif`dir ve tüm katılımcılar, haklarını ticarete açmak için bir `sandalye`ye sahiptirler. Bu nedenle, bir tekliften bir `userSeat` döndüğünü açıklar. 

#### Ama bekleyin, bu iki yüz ne?
`Zoe` çerçevesini, iki ayrı bileşenin toplamı olarak düşünebiliriz;
* `Zoe Contract Facet(ZCF)`: Yalnızca sözleşme geliştiricilere açık olan bir API. İstemci uygulamaların bu API ile etkileşim kurma yolları yoktur.
* `Zoe Service`: Bu API, tüm dünyaya açıktır. İster bir istemci uygulama, ister REPL ile oynayan bir kullanıcı olsun, hepsi bu API'ye erişebilir.

#### Bir `seat` hangi API'ye aittir?
Bir nesnenin hangi API'ye (`ZCF` veya `Zoe Service`) ait olduğunu anlamaya çalıştığınızda, kendinize şu soruyu sorun: Bu nesne, sözleşme içinden senkron bir şekilde erişilebilir mi? Eğer evetse, bu nesneyi [ZCF Dokümanları](https://docs.agoric.com/reference/zoe-api/zoe-contract-facet.html) içinde arayın. Aksi takdirde, [Zoe Service Dokümanları](https://docs.agoric.com/reference/zoe-api/zoe.html) içinde arayın.  

Yukarıdaki ipucunu `userSeat`'e uygulayalım,
* `userSeat`, bir tekliften döndürülür
* Bu `userSeat`'den `offerResult`'u alırız
  * `offerResult`, sözleşme kodunu çalıştırdıktan sonra döndürdüğü şeydir
* `haklarımızı` bu `userSeat`'den çekeriz
* Sözleşme kodu, bu nesneye senkron bir erişime sahip değildir

Sonuç: `userSeat`, `Zoe Service` API'nin bir parçasıdır.

#### Bir sözleşme, `userSeat`'e nasıl hak geçer? İşte burada `zcfSeat` devreye girer!
Her teklifin gerçekleştirilmesi sırasında sözleşmeye verilen `zcfSeat`dir. Kullanıcının ödemelerini bir emanet halinde (sözleşme geliştiricisi erişemez) ve öneriyi içerir, böylece sözleşme geliştiricileri gerekli mantığı zorlayabilir. Varlıklar sözleşme içindeki `zcfSeat`'ler arasında yeniden dağıtıldıktan sonra, `userSeat`'ten çekilebilirler.

### Sözleşme Geliştirme Deneyimi
#### TDD hakkında ne düşünüyorsunuz?
Akıllı Sözleşme Geliştirme, test ortamlarının geliştirici geri bildirim süresi açısından çok maliyetli olduğu bir alandır ve neredeyse TDD yapmak bir `zorunluluk`dur.

#### İşte burada `Ava` devreye girer
Agoric, birim test çerçevesi olarak `ava` kullanır. Geliştirmeye başlamadan önce [dokümantasyonunu kontrol etmek](https://github.com/avajs/ava) yararlı olabilir.

Hadi biraz kod görelim!

## Resources
[1] [Capability Based Financial Instruments](https://papers.agoric.com/assets/pdf/papers/capability-based-financial-instruments.pdf)<br>
[2] [Distributed Electronic Rights in JavaScript](https://papers.agoric.com/assets/pdf/papers/distributed-electronic-rights-in-javascript.pdf)<br>
[3] [Digital Path](https://papers.agoric.com/assets/pdf/papers/digital-path.pdf)
