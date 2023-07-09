# Ders Beş - REPL ve Dağıtım Scriptleri
## İçindekiler
* `home` nesnesi: REPL mi Dağıtım Scriptleri mi?
  * REPL
  * Dağıtım Scripti
* Dapp Geliştiricileri için `home` Nesnesi
  * Nesnelerinizi elde tutmak: `Board` mu `Scratch` mı?
  * NameHub: `agoricNames`, `namesByAddress`
  * `zoe`!
  * Test olmayan bir ortamda teklifler göndermek: `wallet` ve `walletBridge`
  * Zamanlayıcılar: `localTimerService` mı `chainTimerService` mi?
* Demolar
  * CLI İstemcisi olarak Dağıtım Scripti
  * Başka Bir Kullanıcıya Ödemeler Gönderme

## `home` nesnesi: REPL mi Dağıtım Scriptleri mi?

Kullanıcıları olmayan blockchain ağları işe yaramaz. Kullanıcıyı programlar bağlamında temsil eden bir ajan olmalı. Agoric'te bu ajana `ag-solo` denir. Her `ag-solo`'nun kendi `vat`'i vardır. Bu, diğer `vat`'lerde yaşayan bileşenlerle etkileşime girebilmesinin yolu budur. Genel olarak blockchain ekosistemini düşündüğümüzde, ağdaki kullanıcıları temsil eden ajan `wallet` olarak adlandırılır. Agoric'in kendi cüzdan uygulaması da vardır. Ve evet, bu cüzdan `ag-solo` içinde konuşlandırılmıştır.

<img src='images/lectureFive-One.png' width='60%'>

_Şekil 1: Ajanların olası yapılandırmalarını gösterir_

Yukarıdaki diyagramda, kullanıcılar kendi `ag-solo`'larında temsil edilir:
* Ag-solo (Alice)
* Ag-solo (Bob)
* Ag-solo (Sam)

Ve bu kullanıcılar bir blockchain ile etkileşime girer, Agoric Blockchain için olası yapılandırmalar şunlardır:
* Sim-Chain
* Yerel Tek Düğüm Zinciri
* Uzaktan Zincir: Ana ağ veya Test ağı

Agoric'te geçirdiğim zamanı dikkate alırsak, bir dapp geliştirici için en iyi geliştirme deneyimi `Yerel Tek Düğüm Zinciri`dir çünkü kodunuzu gerçek bir düğümde çalıştırabilir ve ağla ilgili sorunlarla engellenmezsiniz.

Takip eden derslerde `Yerel Tek Düğüm Zinciri` nasıl başlatılır üzerine odaklanacağız.


### `home` nesnesi nedir?
`home`, `ag-solo`'nun ağı ve diğer kullanıcılarla etkileşim kurmasına izin veren araçlar için kök nesnedir. Bu `home`
nesnesi iki yerden erişilebilir:
1. REPL
2. Dağıtım Scriptleri

### REPL
**R**ead-**E**val-**P**rint-**L**oop - REPL, cüzdan UI'nızın içine entegre edilmiştir. Hatırlayın ki kursun başından beri cüzdan UI'mıza erişmek için aşağıdaki komutu kullandık:

```sh
agoric open --repl
```

Buradaki `--repl` bayrağı, REPL'yi cüzdan UI'mızda erişilebilir hale getirmek istediğimizi gösterir. İşte REPL'den erişilen `home`
nesnesinin bir ekran görüntüsü:

<img src='images/replHome.png' width='60%'>

`home` nesnesinde yaşayan bazı nesneler `ag-solo` için dahilidir, bazıları kullanımdan kaldırılmıştır ve bazıları
dapp geliştiricileri için çok yararlıdır. Biz son tip üzerine odaklanacağız.

Hangisinin hangisi olduğuna dair tam bir liste için lütfen [REPL Docs](https://docs.agoric.com/reference/repl/) bakınız.

### Dağıtım Scriptleri
Blockchain ekosisteminde, `dağıtım scripti` terimi, geliştiricilerin akıllı sözleşmelerini üzerinde inşa ettikleri blockchain ağına dağıtmak için kullandığı bir scripti belirtir. Agoric'te bu, bir dağıtım scriptinin yalnızca bir özelliğidir. 

**_Dağıtım scriptlerinin `home` nesnesine tam erişimi vardır._**

Hatırlayın ki `home` nesnesi, dapp geliştiricileri için bir dizi yararlı araç içerir. Dolayısıyla, dağıtım scriptlerini akıllı sözleşmeleri dağıtmak dışında başka amaçlar için kullanmamızın önünde bir engel yok. Bu dersin ilerleyen bölümlerinde bir dağıtım scripti ile ne yapabileceğimizi birkaç canlı demo ile keşfedeceğiz. Şimdilik odaklanalım:

**Bir `dağıtım scripti` ne yapar?**

Agoric CLI'nın `deploy` adında bir alt komutu vardır. Bu dağıtım scripti, argüman olarak istediği kadar script kabul eder ve 
onları sırayla çalıştırır. Örneğin;

```shell
agoric deploy scriptOne.js scriptTwo.js scriptThree.js
```

Yukarıdaki örnek komut, tüm scriptleri aşağıdaki sırayla çalıştırır;
1. scriptOne.js 
2. scriptTwo.js 
3. scriptThree.js

Harika. Ama;

**`agoric deploy` herhangi bir JS scriptini çalıştırabilir mi?**

Hayır, aşağıdaki yapıyı takip etmelidir:

```js
const scriptOne = async (homeP, others) => {
  const home = await homeP;
  console.log({ home, others });
};

export default scriptOne;
```

"`agoric deploy` her betiği iki argümanla çağırır;
1. `homeP`: `home` nesnesi için bir promise
2. `others`: Bizim kontratlarımızı paketlemek için kullandığımız `bundleSource` gibi ve kontratlarımıza yol çözümlemek için kullandığımız `pathResolve` gibi bazı host ile ilgili yardımcı nesneler.

**!!! Önemli !!!:** Eğer `agoric deploy` kullanarak bir betiği çalıştırmak isterseniz, çalıştırmak istediğiniz yöntemi `export default` ile dışa aktarmanız gerekir. Yukarıdaki örnekte, çalıştırmak istediğimiz yöntem `scriptOne` olduğu için, son satıra şunu ekliyoruz;

```js
// ...
export default scriptOne;
```

## Dapp Geliştiriciler için `home` Nesnesi
Daha önce de belirttiğimiz gibi `home` nesnesi birçok araç içerir. Bunlardan bazıları dahili kullanımlar içindir, bazıları agoric-sdk'nin geliştirme aşamasında olduğu için kullanımdan kaldırılmıştır ve bazıları dapp geliştiriciler için çok yararlıdır. Bu yardımcı araçları nasıl kullanabileceğimizi keşfedelim.

### Nesnelerinizi koruma: `Board` vs `Scratch`?
`Hardened JavaScript` bir `OCaps` zorlama ortamıdır. Bu, bir nesnenin referansını bir kere kaybettiğinizde, bu yeteneği de kaybettiğiniz anlamına gelir. `home` nesnesi nesneleri saklama için iki seçenek sunar;

1. Board
2. Scratch

Bir senaryoyu düşünelim ki nesneleri tutmak istiyoruz:

```js
const {
  publicFacet,
  creatorFacet,
  instance,
} = await E(zoe).startInstance(
  installation,
);
```

Ve hayali sözleşmemizde `publicFacet` ve `creatorFacet` içeriği diyelim ki şöyle:

```js
const publicFacet = Far('Public Facet', {
  hello: () => 'Hello from Public Facet!!!',
});

const creatorFacet = Far('Creator Facet', {
  dangerous: () => zcf.shutdown(),
});
```

Genellikle `dangerous` gibi güçlü yöntemlerin `creatorFacet`'te olduğunu bekleriz. Bu yüzden bu hayali kontratın yaratıcısı olarak `creatorFacet`'i korumamız gerekiyor, ama aynı zamanda kullanıcılarımızın `publicFacet` ile etkileşimde bulunmasına izin vermeliyiz. Nesnelerimizi saklayabileceğimiz iki ayrı yere ihtiyacımız var gibi görünüyor;
* **Public Objects** için bir yer
* **Private Objects** için bir yer

Agoric bu ihtiyaçlar için iki depolama yeri sunar: `Board` PUBLIC depolama için ve `Scratch` PRIVATE depolama için.

`home` nesnesini hayali senaryomuz için bir örnek deploy betiğinde kullanmak istesek, aşağıdaki gibi görünür:

```js
const scriptOne = async (homeP, { pathResolve }) => {
  const home = await homeP;

  const {
    publicFacet,
    creatorFacet,
    instance,
  } = await E(zoe).startInstance(
    installation
  );
  
  const [publicFacetBoardId, creatorFacetScratchId] = await Promise.all([
    E(home.board).getId(publicFacet),
    E(home.scratch).set('creator_facet_scratch_id', creatorFacet)
  ]);

  const dappConstants = {
    publicFacetBoardId,
    creatorFacetScratchId
  };
  const defaultsFile = pathResolve(`./generated/dappConstants.js`);
  console.log('writing', defaultsFile);
  const defaultsContents = `\
// GENERATED FROM ${pathResolve('./scriptOne.js')}
export default ${JSON.stringify(dappConstants, undefined, 2)};
`;

  await fs.promises.writeFile(defaultsFile, defaultsContents);
};

export default scriptOne;
```

Kodu incelerken, aşağıdaki bölümde `creatorFacet` ve `publicFacet`'i depoya koyuyoruz:

```js
const [publicFacetBoardId, creatorFacetScratchId] = await Promise.all([
    E(home.board).getId(publicFacet),
    E(home.scratch).set('creator_facet_scratch_id', creatorFacet)
  ]);
```


* `E(home.board).getId(publicFacet)` değeri `board`a ekler (eğer zaten mevcut değilse) ve oluşturulan benzersiz id'yi döndürür,
değer zaten varsa sadece id'yi döndürür. `publicFacet`, `board`da saklandığı için KAMU malıdır.
* `E(home.scratch).set('creator_facet_scratch_id', creatorFacet)` bir anahtar-değer çiftini argüman olarak alır, anahtarı döndürür. 
`creatorFacet`, `scratch`da saklandığı için ÖZEL'dir.

Dokümantasyonu aşağıdaki linklerden inceleyebilirsiniz;
* [Board API Referansı](https://docs.agoric.com/reference/repl/board.html)
* [Scratch Referansı](https://docs.agoric.com/reference/repl/scratch.html#e-home-scratch-set-id-obj)

Nesnelerimizi depoya koyduktan sonra, anahtarları güvenli bir yerde saklamalıyız. Çoğunlukla, bu yerel bir dosyaya yazmak anlamına gelir. 
Yukarıdaki scriptte, dosya adı `dappConstant.js`dir. Bu yerel dosyaya erişimi olan herkes, `Deploy Script`leri ve `REPL`'in aynı `home` 
nesnesini paylaştığı için bu nesneleri REPL'den sorgulayabilir. Örneğin, `publicFacet` için tahta id'si 'board0371' olsun. `publicFacet`, 
**herhangi bir** `ag-solo` tarafından aşağıdaki gibi sorgulanabilir:

<img src="images/replBoardQ.png" width='40%'>

Yalnızca `agoric deploy`'a bağlı olan `ag-solo`, `home.scratch`'ın özel olmasından dolayı `creatorFacet`'i sorgulayabilir:

<img src="images/replScratchQ.png" width='40%'>

### NameHub: `agoricNames`, `namesByAddress`
`NameHubKit`, kullanıcıların isim hiyerarşisine dayalı olarak nesneleri saklamasına/ sorgulamasına olanak sağlayan, Agoric ekibi 
tarafından uygulanan bir veri yapısıdır.

Bir `NameHubKit`'in iki bileşeni vardır;
* Veri okumak için `NameHub`
* Yazma erişimi için `NameAdmin`

`NameHubKit`'in tam liste metotları, açıklamaları ve kaynak kodunu `NameHubKit Dokümantasyonu` bölümünden incelemeyi unutmayın.

`home` nesnesi, iki `NameHub` örneği sunar: `agoricNames` ve `namesByAddress`,

Ve bir `NameAdmin` örneği: `myAddressNameAdmin`, `namesByAddress` için `NameAdmin`'dir.

#### `agoricNames`
Normal kullanıcıların `agoricNames`'e yazma erişimi olmadığını fark edin. Neden?

`agoricNames`, kullanıcının çekirdek Agoric bileşenlerine erişim elde edebileceği bir `NameHub` özel örneğidir.
Herhangi bir kullanıcının bu kadar önemli bir nesneye yazma izni verilmesi, tüm ağ için bazı güvenlik sorunlarına neden olabilir, 
bu yüzden kullanıcılar `agoricNames` için bir `NameAdmin` alamaz.

**`agoricNames` içinde neler var?**

`agoricNames`'in içeriğini görmek istersek, bunu yapabiliriz:

<img src="images/replAgoricNames.png" width='60%'>

Bir iç içe geçmiş dizi döndürüldüğünü fark edin, iç dizinin ilk elemanı `anahtar` ve ikinci eleman bir `nameHub` örneğidir. 
Bu, aşağıdaki gibi bir şey yapabileceğimiz anlamına gelir:

<img src="images/replLookupIssuer.png" width='60%'>

Ve `agoricNames`'i sorguladığımız aynı şekilde döndürülen `nameHub`'ı sorgulayabiliriz:

<img src="images/replIssuerEntries.png" width='60%'>

`lookup` metodu, argümanlar olarak bir yol belirlemenize de izin verir, bu yüzden aşağıdaki gibi bir şey mümkündür:

<img src="images/replLookupBld.png" width='60%'>

`lookup`'a 3 veya 4 argüman geçirmek mümkündür, burada iç içe geçmiş dizinin seviyesi bu kadar derindir.

#### namesByAddress
`namesByAddress` de bir `NameHub` örneğidir ancak ağ genelinde çekirdek bileşenlere erişim için değil, kullanıcının kişisel 
depolaması için. `myAddressNameAdmin`'dan kendi ag-solo'nuz için `NameAdmin`'e erişebilirsiniz. Bir örnek senaryo üzerinden gidelim;

> **Önemli:** Burada not edilmesi gereken önemli bir şey, kullanıcıların tüm diğer kullanıcıların `NameHub`larını görebilmeleri ancak 
> sadece kendi `NameHub`larına yazabilmeleridir. Bu, `NameHub`'ların iç içe geçmiş yapısı sayesinde mümkündür.

Alice'nin, yalnızca Bob ile paylaşmak istediği önemli bir nesnesi olduğunu hayal edin. Agoric bir OCaps ortamı olduğundan 
bu çok olası bir senaryodur. Bu önemli nesnenin referansını Bob ile nasıl paylaşabilir?

Bu amaç için uygulanmış bir akıllı kontratın olduğunu varsayalım, ki bu da çok olasıdır, kaynak kodu aşağıdaki gibi olsun:

```js
const start = () => {
  const importantObjects = [];

  const creatorFacet = Far('Important Object Receiver', {
    receiveNew: newImportantObject => {
      importantObjects.push(newImportantObject

Aşağıdaki kodun Türkçe çevirisi;

```js
const start = () => {
  const importantObjects = [];

  const creatorFacet = Far('Önemli Nesne Alıcı', {
    receiveNew: yeniÖnemliNesne => {
      importantObjects.push(yeniÖnemliNesne);
    },
    readImportantObjects: () => {
      return [...importantObjects];
    }
  });

  return { creatorFacet };
};
```

Yukarıdaki kontratta yalnızca `creatorFacet`'i açığa çıkarıyoruz çünkü sadece Bob'un adresini bilen kişilerin Bob ile nesneleri 
paylaşabilmelerini sağlamak istiyoruz. Bob bu kontratı başlatacak ve `creatorFacet`'i `nameHub`'ına koyacak. Bu işlemi tek seferde 
bir dağıtım scripti ile yapabilir. Bir dağıtım scripti örneği aşağıdaki gibi olabilir:

```js
const shareImportantObject = async (homeP , endowments) => {
  const { myAddressNameAdmin, zoe } = E.get(homeP);
  const { install } = await makeHelpers(homeP, endowments);

  console.log('Installting objectReceiver contract...');
  const { installation } = await install(
    '../../contract/src/lectureFive/objectReceiver.js',
    'Object Receiver Contract'
  );

  console.log('Starting objectReceiver contract...');
  const {
    creatorFacet: objectReceiverCreatorFacet
  } = await E(zoe).startInstance(
    installation
  );

  console.log('Putting objectReceiverCreatorFacet to namesByAddress...');
  await E(myAddressNameAdmin).update('objectReceiver', objectReceiverCreatorFacet);

  console.log('Done.');
};

export default shareImportantObject;
```
**Not:** [Dağıtım betiğinin kaynak kodu.](../codeSamples/api/lectureFive/shareImportantObject.js)

**Dikkat:** `E.get()` bir Promise'den nesne çekmeyi, çözülmesini beklemeksizin sağlar.

**Dağıtım betiği ne yapıyor?**

Kaynak koddaki **23. satıra** kadar olan kod, kontratın yüklenmesi ve örneğin başlatılması içindir. 
[deploy-script-support](https://github.com/Agoric/agoric-sdk/tree/65d3f14c8102993168d2568eed5e6acbcba0c48a/packages/deploy-script-support) başlığını kontrol edin.

İlgilendiğimiz kısım:

```js
console.log('NesneAlıcıYaratıcıFacet isimliAdreslere ekleniyor...');
await E(myAddressNameAdmin).update('objectReceiver', objectReceiverCreatorFacet);
```

Dikkat edin;
* Dağıtım betiği `myAddressAdmin` ile aynı `vat`'te olmadığı için [eventual-send](https://github.com/endojs/endo/tree/master/packages/eventual-send)(E)'yi kullanıyoruz.
* Yeni bir nesneyi başlatmak için `update` metodu kullanılır.
* `console.log` aracılığıyla Bob'a ne yaptığımızı bildiriyoruz.

Yukarıdaki betik çalıştırıldığında, Bob'un adresini bilen herkes onunla nesneleri paylaşabilir. REPL'den paylaşılan nesneyle etkileşim kurmak aşağıdaki gibi görünebilir:

> **Not:** Bir simülasyon zinciri başlattım, dolayısıyla kullanıcının adresi `sim-chain-client`'ta.

> **Not:** Simülasyon zincirindeyiz, bu yüzden hem Bob hem de Alice gibi davranıyoruz.

`objectReceiver`'ı sorgulayın;

<img src="images/lookupobjectReceiver.png" width='60%'>

İçerikleri kontrol edin;

<img src="images/firstReadOR.png" width='60%'>

`topSecret` oluşturun ve Bob'a gönderin;

<img src="images/writeOR.png" width='60%'>

`topSecret`'ı alın ve füzeleri fırlatın;

<img src="images/readAgainOR.png" width='60%'>


Bu şekilde diğer kullanıcılara özel bir şekilde nesneler gönderebilirsiniz. Canlı demoda diğer kullanıcılara nasıl ödeme gönderileceğini göstereceğiz.

#### NameHubKit Dokümantasyonu
* [NameHub API Referansı](https://github.com/Agoric/agoric-sdk/blob/65d3f14c8102993168d2568eed5e6acbcba0c48a/packages/vats/src/types.js#L14-L30)
* [NameAdmin API Referansı](https://github.com/Agoric/agoric-sdk/blob/65d3f14c8102993168d2568eed5e6acbcba0c48a/packages/vats/src/types.js#L32-L57)
* [NameHubKit Kaynak Kodu](https://github.com/Agoric/agoric-sdk/blob/65d3f14c8102993168d2568eed5e6acbcba0c48a/packages/vats/src/nameHub.js)

### `zoe`!
Hatırlayın ki `zoe`'nin iki yüzü vardır;
1. `ZoeService` çoğunlukla müşteri amaçları içindir
2. `ZoeContractFacet` akıllı kontrat geliştiricileri için bir araç seti sunar

REPL ve Dağıtım Betikleri bağlamında düşündüğümüzde, bizim müşteri tarafında olduğumuz açıktır. Bu, `home` nesnesinin bize `zoe` olarak sunduğu şeyin aslında `ZoeService` olduğu anlamına gelir. Tüm [ZoeService API Referansı](https://docs.agoric.com/reference/zoe-api/zoe.html)'na göz atmakta özgürsünüz. `ZoeService` API'deki tüm metodlar hem REPL hem de Dağıtım Betiklerinde çalışır. Ben sadece sık kullanılan birkaç metoda değinmek istiyorum:

<img src="images/chainboardDiagrams-lecture-five-zoe.png" width='60%'>

_Figure 2: Zoe'nin Dağıtım Betiklerinde ve REPL'de sıkça kullanılan metodları gösterir_

`Zoe`, Agoric Ağı için kontrat hostudur ve bu hosta konuşlandırılan tüm kontratlar aynı şekilde yapılmalıdır. İki ana metot vardır:

1. `E(home.zoe).install()`
2. `E(home.zoe).startInstance()`


Aşağıdaki kodları ve açıklamalarını satır satır Türkçe'ye çevirelim:

Bir sözleşmeyi başarıyla yüklediğimiz ve örneklendirdiğimizde, sözleşmenin örneğine referansla birkaç işlem yapabiliriz:

1. `E(home.zoe).getBrands(instance)` ve `E(home.zoe).getIssuers(instance)` 
2. `E(home.zoe).getTerms(instance)`
3. `E(home.zoe).getPublicFacet(instance)`

Tüm [ZoeService API Referansı'nı](https://docs.agoric.com/reference/zoe-api/zoe.html) incelemeyi unutmayın.

### Test olmayan bir ortamda teklif gönderme: `wallet` ve `walletBridge`
Teklif Güvenliği, Agoric'in sunduğu değer açısından en önemli yönlerden biridir.
Şimdiye kadar teklif göndermenin sadece bir yolunu gördük, o da;

```js
const userSeat = await E(zoe).offer(
  invitation,
  proposal,
  payment,
  offerArgs, // İsteğe bağlı
);
```

* **invitation:** `offerHandler`a bir referans tutar, gerekli
* **proposal:** Kullanıcı ne istediğini ve karşılığında ne ödemeye hazır olduğunu belirtir, teklifte bir hak transferi olacaksa gerekli
* **payment:** Kullanıcının gerçekten ödediği varlık, teklifte `give` özelliği varsa gerekli
* **offerArgs:** İş mantığına özgü veri, isteğe bağlı

Bazı `Moola` fungible varlıklarını, belirli bir miktar `Quatloos` fungible varlık karşılığında almak istediğimiz bir senaryo hayal edelim. Aşağıdaki gibi bir birim testinde ilgili teklifi gerçekleştirebiliriz:

```js
const {
  brand: quatloosBrand,
  mint: quatloosMint
} = makeIssuerKit('Quatloos');

const quatloosAmount = AmountMath.make(quatloosBrand, 100n);
const moolaAmount = AmountMath.make(moolaBrand, 200n);

const proposal = {
  give: { Price: quatloosAmount },
  want: { Asset: moolaAmount }
};

const payment = {
  Price: quatloosMint.mintPayment(quatloosAmount),
};

const userSeat = await E(zoe).offer(
  E(publicFacet).makeAssetInvitation(),
  proposal,
  payment,
);
```

Gereken ödemeyi `quatloosMint.mintPayment(quatloosAmount)` çağırarak oluşturduğumuzu fark edin. Bu, gerçek bir ağ ortamında mümkün olabilir mi? Hayır, mümkün değil. Eğer mümkün olsaydı, bu paranın basıldığı anlamına gelirdi. Bunu engelleyen teknik bir sebep olmasa da, basabileceğiniz bir varlığı insanların alışveriş yapacak olmaları çok uzak bir ihtimal. Peki, ödeme nasıl elde edilir?

**İşte burada `home.wallet` devreye girer**

Daha önceki bölümlerde elektronik bir hakkı, ERTP arayüzünü uygulayarak bir e-hakkına nasıl dönüştürebileceğimizden bahsetmiştik. ERTP şunları içerir;

* Emitent (Issuer)
* Para Basımı (Mint)
* Cüzdan (Purse)

Ve liste böyle devam eder. Tam belgelendirme için [ERTP API Referansı'nı](https://docs.agoric.com/reference/ertp-api/) gözden geçirin.
`wallet` nesneleri, kullanıcı açısından ERTP API'nın en önemli bileşenlerine erişim sağlar. Özellikle;
* **Purse (Cüzdan):** Ödemelerin saklandığı yer.
* **Payment (Ödeme):** Gerçek para.

İlgili metotlar aşağıdaki gibidir;
* `E(home.wallet).addPayment(payment)`
* `E(home.wallet).getIssuers()`
* `E(home.wallet).getIssuer(petname)`
* `E(home.wallet).getPurses()`
* `E(home.wallet).getPurse(petname)`
  
> Note: [Definition of Petname](https://docs.agoric.com/glossary/#petname)

See [Wallet API Reference](https://docs.agoric.com/reference/wallet-api.html#wallet-api-commands) for full list of 
methods.

Let's try to send this desired offer from a deploy script using `home.wallet` object;

```js
/**
 * We assume;
 * - All board ids are already known
 * - All petnames are known
 */
const sendOfferUsingWalletAPI = async homeP => {
  const { zoe, board, wallet } = E.get(homeP);
  const quatloosPursePetname = 'Quatloos Purse';
  const moolaPursePetname = 'Moola Purse';
  
  // Get values from board
  const [
    quatloosBrand,
    moolaBrand,
    publicFacet, // Public Facet of imaginary contract that is going to do the trade
  ] = await Promise.all([
    E(board).getValue(QUATLOOS_BRAND_BOARD_ID),
    E(board).getValue(MOOLA_BRAND_BOARD_ID),
    E(board).getValue(PUBLIC_FACET_BOARD_ID),
  ]);
  
  // Build offer
  const quatloosAmount = AmountMath.make(quatloosBrand, 100n);
  const moolaAmount = AmountMath.make(moolaBrand, 200n);
  
  const proposal = harden({
    give: { Price: quatloosAmount },
    want: { Asset: moolaAmount }
  });
  
  // Get purses
  const quatloosPurseP = E(wallet).getPurse(quatloosPursePetname);
  const moolaPurseP = E(wallet).getPurse(moolaPursePetname);
  
  // Get payment
  const quatloosPayment = await E(quatloosPurseP).withdraw(quatloosAmount);
  
  const payment = harden({
    Price: quatloosPayment
  });
  
  const userSeat = E(zoe).offer(
    E(publicFacet).makeAssetInvitation(),
    proposal,
    payment
  );
  
  // Make sure offer is successful, this will throw if there was an uncatched error when executing the offer
  await E(userSeat).getOfferResult();
  
  // Get payout
  const moolaPayment = await E(userSeat).getPayout('Asset');
  
  // Put moola into purse
  await E(moolaPurseP).deposit(moolaPayment);
};

export default sendOfferUsingWalletAPI;
```

Let's break down the above code step-by-step;
1. We fetch references to the required objects from the `board`
2. We create the proposal with `Amounts`
3. We fetch purses to the desired assets from our wallet
4. We withdraw from `quatloosPurse` the amount of `quatloos` we specified in our `give` section of the proposal
5. We build the payment
6. We send the offer
7. Check everything went alright
8. Withdraw what we wanted from the `userSeat`
9. Deposit the withdrawn `moola` to the `moolaPurseP`

Can you sense some anti-pattern in the above code? What happened to the "No developers should get their hands on 
to any payment in the code"? Yeah, something's wrong here. Although this might be a useful approach for demos, this
is not a production approach. So what is the solution then?

**Enter `walletBridge`**

`walletBridge` provides an API for untrusted dapps to let them interact with the wallet and therefore, the assets
in the wallet. See [WalletBridge API Reference](https://docs.agoric.com/reference/wallet-api.html#walletbridge-api-commands)
for full list of available methods. 

In Web3, almost every dapp asks for user to connect their wallet. In Agoric, this is also the case. Dapps ask for
approval from users so that they can do their job. Approving a dapp means;
* A `walletBridge` instance specific to that dapp is created and returned to the dapp
* User's wallet now knows that dapp

The term `bridge` is used because it sits between the full features of `home.wallet` and the dapp. Every offer
sent via `walletBrdige` is subject to user approval. These offer requests show up on the Wallet UI's `Dashboard`.
The end-to-end flow can be seen in the below diagram:

```mermaid
sequenceDiagram
   actor User
   participant CU as Dapp
   participant W as Wallet
   User->>CU: Opens the dapp page
   CU->>+W: Tries to connect to ag-solo
   W ->>+ User: Asks for user approval
   User->>-W: Approves the dapp
   W-->>-CU: Returns a WalletBridge object
   CU->>+W: E(walletBridge).addOffer(offerConfig)
   W ->>+ User: Asks for user approval
   User->>-W: Approves the offer
   W -->>-CU: Returns offerID
```

Now, let's try to update the anti-pattern code into a safe CLI client to our wallet:

```js
/**
 * We assume;
 * - All board ids are already known
 * - All petnames are known
 */
const safeOfferClient = async homeP => {
  const { board, wallet } = E.get(homeP);
  const quatloosPursePetname = 'Quatloos Purse';
  const moolaPursePetname = 'Moola Purse';
  
  const publicFacetP = E(board).getValue(PUBLIC_FACET_BOARD_ID);
  const walletBridge = E(wallet).getBridge();
  
  const offerConfig = {
    id: `${Date.now()}`,
    invitation: E(publicFacetP).makeBorrowInvitation(),
    installationHandleBoardId: IMAGINARY_CONTRACT_INSTALL_BOARD_ID,
    instanceHandleBoardId: IMAGINARY_CONTRACT_INSTANCE_BOARD_ID,
    proposalTemplate: {
      want: {
        Asset: {
          pursePetname: moolaPursePetname,
          value: 200n,
        },
      },
      give: {
        Price: {
          pursePetname: quatloosPursePetname,
          value: 100n,
        },
      }
    },
  };
  
  await E(walletBridge).addOffer(offerConfig);
  console.log('Offer sent.');
};

export default safeOfferClient;
```

Once we execute this script with `agoric deploy`, the code above will send the exact offer as we sent in the 
earlier sample. Notice;
* There are not `payments` flying around
* OfferHandler will be executed after the user approves offer request from Wallet UI

One more important thing to mention:

```js
const walletBridge = E(wallet).getBridge();
```

In the above line, we fetched the `walletBridge` using `getBridge` method. In the documentation, you'll also see a
`getScopedBridge` method which is what a dapp will use in a real world scenario. The difference between `getBridge` 
and `getScopedBridge` is that `E(wallet).getScopedBridge()` requires a user approval whereas
`E(wallet).getBridge()` does not. Since deploy scripts are considered trusted agents we were able to use 
`getBridge`, if we were a dapp living in a browser, and try to fetch a `walletBridge` instance from the wallet
we would have to use `getScopedBridge` instead.

**Deploy Scitps Can Mimic Client Apps**
With the usage of `wallet` you can test your contracts in a non-unit-test-environment without building a GUI.
A very useful feature for demos.

### Timers: `localTimerService` vs `chainTimerService`?
Agoric offers a shared time service across all agents of the network. Any parties doing a business that depends on time
can use this service to make sure all time passes the same for everybody. 

This shared time service is called `chainTimerService`. Agoric also offers a `localTimerService` which is mainly used
for tests. These timer services both share an identical API. The only difference between them is the _unit of time_:

* `localTimerService` interprets time **milliseconds**.
* `chainTimerService` interprets time **seconds**.

[See the full docs on timer services.](https://docs.agoric.com/reference/repl/timerServices.html#timerservice-objects)

When you're developing a smart contract that depends on time, the usual way to go is to accept a `timer` from the 
contract terms:

```js
const start = async zcf => {
  const {
    timer,
  } = zcf.getTerms();
};

harden(start);
export { start };
```

And in your deploy script you can inject any timer service as you like to your contract:

```js
const deploy = async homeP => {
  const { zoe, localTimerService, chainTimerService } = E.get(homeP);
  
  const sampleTermsOne = {
    timer: localTimerService
  };
  
  const sampleTermTwo = {
    timer: chainTimerService,
  };
  
  await E(zoe).startInstance(
    installation,
    issuerKeywordRecord,
    sampleTermsOne      
  );

  await E(zoe).startInstance(
    installation,
    issuerKeywordRecord,
    sampleTermsTwo
  );
};

export default deploy;
```

Both terms are valid and can be used by your contract.

There's a [manualTimer](https://github.com/Agoric/agoric-sdk/blob/65d3f14c8102993168d2568eed5e6acbcba0c48a/packages/zoe/tools/manualTimer.js)
that is very useful for testing as well. Make sure to check this out.



## Demo Time!
* Deploy Script as a CLI Client
* Sending invitation to other users privately 
