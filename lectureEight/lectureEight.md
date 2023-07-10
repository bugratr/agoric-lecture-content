# Sekizinci Ders - Agoric Önceden Oluşturulmuş Sözleşmeleri & Fiyat Otoritesi

## İçerik Tablosu

- Önceden oluşturulmuş sözleşmeler
    - Finanse Edilmiş Call Spread (Opsiyonlar Stratejisi)
    - Basit Borsa
- Fiyat Otoritesi
    - Fiyat Teklifi
    - Fiyat Teklifi Bildiricisi
    - Anlık Fiyat Teklifleri
        - VerilenTeklif
        - ArananTeklif
    - Söz Verilen Fiyat Teklifi
        - NeZamanTeklif
        - DeğiştirilebilirNeZamanTeklif
    - Manuel Fiyat Otoritesi

## Agoric Önceden Oluşturulmuş Sözleşmeleri
 
Agoric, geliştiricilerin uygulamalarını oluştururken yardımcı olmak için oluşturulan [Agoric Önceden Oluşturulmuş Sözleşmeleri](https://github.com/Agoric/documentation/blob/24b31f8e49cc67d05e229992103bbecc0b2e5ead/main/guides/zoe/contracts/README.md) adlı bir liste sunar. Bu sözleşmeler, Agoric'in `yüksek düzeyli (high-order)` özelliklerinden yararlanarak bir ana sözleşme aracılığıyla örneklenmelidir.
Daha fazla bilgi için [Dördüncü Ders](...)'i incelemenizi öneririz.

İki farklı paketten seçtiğimiz iki sözleşme, bu sözleşmelerin işlevselliğini ve nasıl kullanılacağını göstermektedir.

## Finanse Edilmiş Call Spread

`DeFi paketi`nden `finanse edilmiş call spread` sözleşmesi, tamamen teminatlandırılmış bir call spread'i uygular.
Bir `call spread`'inde yatırımcı, `daha düşük bir kullanım fiyatı`na sahip bir `call opsiyonu` satın alır (bu "uzun call" olarak adlandırılır) ve `daha yüksek bir kullanım fiyatına` sahip başka bir call opsiyonu satar (bu "kısa call" olarak adlandırılır). Kullanım fiyatlarındaki fark, bir `spread` oluşturur ve stratejinin amacı, iki kullanım fiyatı arasındaki farktan kar etmektir. Uzun call, sınırlı bir yukarı yönlü potansiyel sağlar, kısa call ise gelir üretir ve uzun call'ın maliyetini karşılamaya yardımcı olur. Bir call spread'de satılan ve alınan iki opsiyonun aynı `vade tarihine` sahip olması gerekir.

Bir `call opsiyonu`, sahibine belirli bir fiyat (kullanım fiyatı) karşılığında belirli bir zaman dilimi içerisinde bir varlığı (örneğin bir hisse senedi, emtia veya para birimi) satın alma hakkı veren, ancak *zorunluluğu olmayan* bir finansal sözleşmedir. Call opsiyonları, yatırım ve spekülasyon formu olarak, aynı zamanda hedging (riskten korunma) amacıyla da kullanılır. Eğer dayanak varlığın fiyatı, kullanım fiyatının üzerine çıkarsa, call opsiyonunun sahibi opsiyonu kullanmayı seçebilir ve varlığı daha düşük kullanım fiyatından satın alabilir, bu da bir kar anlamına gelir. Eğer fiyat, kullanım fiyatının üzerine çıkmazsa, opsiyon değersiz hale gelir ve opsiyon için ödenen prim kaybedilir.

Daha fazla bilgi için, aşağıdakileri okumanızı öneririz:

- [Dokümantasyon](https://github.com/Agoric/documentation/blob/24b31f8e49cc67d05e229992103bbecc0b2e5ead/main/guides/zoe/contracts/fundedCallSpread.md)
- [Sözleşme kodu](https://github.com/Agoric/agoric-sdk/blob/4e0aece631d8310c7ab8ef3f46fad8981f64d208/packages/zoe/src/contracts/callSpread/fundedCallSpread.js)
- [Sözleşme testi](https://github.com/Agoric/agoric-sdk/blob/4e0aece631d8310c7ab8ef3f46fad8981f64d208/packages/zoe/test/unitTests/contracts/test-callSpread.js)
- [Call Spread video](https://www.youtube.com/watch?v=m5Pf2d1tHCs&t=3989s)

Ana sözleşmenizde finanse edilmiş call spread sözleşmesi `kurulumu`na erişiminiz olduğunu varsayarsak, `startInstance` fonksiyonunu çağırmak için gereken argümanlar `issuerKeywordRecord` ve `terms`dir.

```js
const { creatorInvitation } = await E(zoe).startInstance(
  installation,
  issuerKeywordRecord,
  terms
);
```

IssuerKeywordRecord, `Dayanak Varlık, Teminat ve Kullanım Fiyatı varlıkları`nın `Düzenleyicisi`ni belirtir.
Dayanak Varlık ve Teminat, aynı Düzenleyiciye sahip olabilir ve Kullanım Fiyatı farklı bir Düzenleyiciye sahip olabilir, veya uygulamanızın bağlamına bağlı olarak üç farklı Düzenleyiciye sahip olabilirsiniz.
Dayanak varlık/kullanım fiyatı çiftinin `fiyat otoritesi`nin Düzenleyicisi de issuerKeywordRecord'a dahildir.

```js
const issuerKeywordRecord = harden({
  Underlying: simoleanIssuer,
  Collateral: bucksIssuer,
  Strike: moolaIssuer,
  Quote: await E(priceAuthority).getQuoteIssuer(
    brands.get("simoleans"),
    brands.get("moola")
  ),
});
```

`issuerKeywordRecord`, `simoleanIssuer` olarak belirlenen `Underlying`, `bucksIssuer` olarak belirlenen `Collateral`, `moolaIssuer` olarak belirlenen `Strike` ve `priceAuthority`'den alınan `Quote` olmak üzere belirli `Issuer`'ları belirler.

`terms` tarafına gelince, bunlar aşağıdakileri içerir:

- `expiration`: Opsiyonun kullanılabilir olduğu süreyi belirler.
- `priceAuthority`: Süre sonunda, temel varlığın değerini strike para biriminde veren bir Fiyat Teklifi çıkarır.
- `underlyingAmount`: Takip edilen varlığın fiyatı.
- `strikePrice1` ve `strikePrice2`: Sırasıyla kısa ve uzun olarak tanımlanan değerler.
- `settlementAmount`: Opsiyonların sahipleri arasında bölünen ve fon sağlayıcı tarafından yatırılan teminat miktarı.
- `timer`: `priceAuthority` tarafından tanınan zamanlayıcı.

```js
const terms = harden({
  expiration: 2n,
  underlyingAmount: simoleans(2n),
  priceAuthority,
  strikePrice1: moola(60n),
  strikePrice2: moola(100n),
  settlementAmount: bucks(300n),
  timer: manualTimer,
});
```

Ev sahibi sözleşme, `LongOption` ve `ShortOption` adında iki opsiyonu geri almak için fundedCallSpread'e bir `teklif` yapar.
Teklifi oluşturmak için, startInstance() çağrıldığında geri dönen `creatorInvitation` ile birlikte, `Proposal` ve `Payment` de gereklidir.

```js
const aliceSeat = await E(zoe).offer(
  creatorInvitation,
  aliceProposal,
  alicePayments
);
```

CreatorInvitation, `makeFundedPairInvitation` fonksiyonu için davetiye ile birlikte, `getInvitationDetails` metodu ile erişilebilen bir `özel obje` döndürür. Bu obje, belirlenen uzun ve kısa miktarları içerir. 
Bu değerler teklif teklifinde `want` özelliği olarak kullanılır. `give` özelliği söz konusu olduğunda, bu, sözleşme şartlarında `settlementAmount` olarak tanımlanan değerden oluşur.

```js
const invitationDetail = await E(zoe).getInvitationDetails(creatorInvitation);
const longOptionAmount = invitationDetail.longAmount;
const shortOptionAmount = invitationDetail.shortAmount;

const aliceProposal = harden({
  want: { LongOption: longOptionAmount, ShortOption: shortOptionAmount },
  give: { Collateral: bucks(300n) },
});
```

Teklifin `ödemesi`, genellikle, teklif teklifinin `give` özelliğinde tanımlanan miktarın basılmış ödemesidir. Bu durumda, spread çağrısının `teminatı`.

```js
const aliceBucksPayment = bucksMint.mintPayment(bucks(300n));
const alicePayments = { Collateral: aliceBucksPayment };
```

Teklif yapıldığında, iki opsiyon pozisyonunu içeren bir ödeme döndürülür.
Pozisyonlar, ücretsiz olarak kullanılabilen davetiyelerdir ve Collateral anahtar kelimesi altında opsiyon ödemelerini sağlarlar.

```js
const { LongOption, ShortOption } = await aliceSeat.getPayouts();
```

Bu iki davetiye, ev sahibi sözleşmeniz tarafından yönetilebilir ve uygulamanızın bağlamına bağlı olarak birçok farklı finansal enstrüman oluşturmanıza olanak sağlar.

## Basit Takas

`simpleExchange` sözleşmesi, `Generic Sales/Trading Contracts package`'ten, ikinci bir varlığı fiyat olarak kullanarak bir varlığı ticaret için basit bir değişimdir.
`order book`, yeni bir sipariş oluşturulduğunda eşleşen `satış` ve `alış` siparişlerini arayan iki basit liste oluşturur. Herkes sipariş oluşturabilir veya doldurabilir ve mevcut sipariş listesini görmek için bir bildirim sistemi vardır.

Daha fazla ayrıntı için, şunları okumanızı öneririz:

```js
const issuerKeywordRecord = harden({
  Underlying: simoleanIssuer,
  Collateral: bucksIssuer,
  Strike: moolaIssuer,
  Quote: await E(priceAuthority).getQuoteIssuer(
    brands.get("simoleans"),
    brands.get("moola")
  ),
});
```

`issuerKeywordRecord`, `simoleanIssuer` olarak belirlenen `Underlying`, `bucksIssuer` olarak belirlenen `Collateral`, `moolaIssuer` olarak belirlenen `Strike` ve `priceAuthority`'den alınan `Quote` olmak üzere belirli `Issuer`'ları belirler.

`terms` tarafına gelince, bunlar aşağıdakileri içerir:

- `expiration`: Opsiyonun kullanılabilir olduğu süreyi belirler.
- `priceAuthority`: Süre sonunda, temel varlığın değerini strike para biriminde veren bir Fiyat Teklifi çıkarır.
- `underlyingAmount`: Takip edilen varlığın fiyatı.
- `strikePrice1` ve `strikePrice2`: Sırasıyla kısa ve uzun olarak tanımlanan değerler.
- `settlementAmount`: Opsiyonların sahipleri arasında bölünen ve fon sağlayıcı tarafından yatırılan teminat miktarı.
- `timer`: `priceAuthority` tarafından tanınan zamanlayıcı.

```js
const terms = harden({
  expiration: 2n,
  underlyingAmount: simoleans(2n),
  priceAuthority,
  strikePrice1: moola(60n),
  strikePrice2: moola(100n),
  settlementAmount: bucks(300n),
  timer: manualTimer,
});
```

Ev sahibi sözleşme, `LongOption` ve `ShortOption` adında iki opsiyonu geri almak için fundedCallSpread'e bir `teklif` yapar.
Teklifi oluşturmak için, startInstance() çağrıldığında geri dönen `creatorInvitation` ile birlikte, `Proposal` ve `Payment` de gereklidir.

```js
const aliceSeat = await E(zoe).offer(
  creatorInvitation,
  aliceProposal,
  alicePayments
);
```

CreatorInvitation, `makeFundedPairInvitation` fonksiyonu için davetiye ile birlikte, `getInvitationDetails` metodu ile erişilebilen bir `özel obje` döndürür. Bu obje, belirlenen uzun ve kısa miktarları içerir. 
Bu değerler teklif teklifinde `want` özelliği olarak kullanılır. `give` özelliği söz konusu olduğunda, bu, sözleşme şartlarında `settlementAmount` olarak tanımlanan değerden oluşur.

```js
const invitationDetail = await E(zoe).getInvitationDetails(creatorInvitation);
const longOptionAmount = invitationDetail.longAmount;
const shortOptionAmount = invitationDetail.shortAmount;

const aliceProposal = harden({
  want: { LongOption: longOptionAmount, ShortOption: shortOptionAmount },
  give: { Collateral: bucks(300n) },
});
```

Teklifin `ödemesi`, genellikle, teklif teklifinin `give` özelliğinde tanımlanan miktarın basılmış ödemesidir. Bu durumda, spread çağrısının `teminatı`.

```js
const aliceBucksPayment = bucksMint.mintPayment(bucks(300n));
const alicePayments = { Collateral: aliceBucksPayment };
```

Teklif yapıldığında, iki opsiyon pozisyonunu içeren bir ödeme döndürülür.
Pozisyonlar, ücretsiz olarak kullanılabilen davetiyelerdir ve Collateral anahtar kelimesi altında opsiyon ödemelerini sağlarlar.

```js
const { LongOption, ShortOption } = await aliceSeat.getPayouts();
```

Bu iki davetiye, ev sahibi sözleşmeniz tarafından yönetilebilir ve uygulamanızın bağlamına bağlı olarak birçok farklı finansal enstrüman oluşturmanıza olanak sağlar.

## Basit Takas

`simpleExchange` sözleşmesi, `Generic Sales/Trading Contracts package`'ten, ikinci bir varlığı fiyat olarak kullanarak bir varlığı ticaret için basit bir değişimdir.
`order book`, yeni bir sipariş oluşturulduğunda eşleşen `satış` ve `alış` siparişlerini arayan iki basit liste oluşturur. Herkes sipariş oluşturabilir veya doldurabilir ve mevcut sipariş listesini görmek için bir bildirim sistemi vardır.

Daha fazla ayrıntı için, şunları okumanızı öneririz:

- [PriceAuthority Zoe documentation](https://github.com/Agoric/documentation/blob/main/main/guides/zoe/price-authority.md)
- [PriceAuthority REPL documentation](https://github.com/Agoric/documentation/blob/main/main/guides/zoe/price-authority.md)
- [PriceAuthority Methods](https://github.com/Agoric/documentation/blob/main/main/reference/zoe-api/contract-support/price-authority.md)

## Fiyat Teklifi Bildirimi

[makeQuoteNotifier](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/src/contractSupport/priceAuthority.js#L244) fonksiyonu, altta yatan bir bildiriciyi kapsayan bir `quote bildirici` oluşturur. Quote bildiricinin amacı, iki varlık arasındaki oranın fiyat tekliflerinin zamanla güncellenen bir akışını sağlamaktır.

MakeQuoteNotifier() iki parametre alır: `amountIn` ve `brandOut`. AmountIn markası ve brandOut, assertBrands fonksiyonunu kullanarak desteklenen çiftin kontrol edilmesi için kontrol edilir.

Daha sonra, altta yatan bildiricinin `getUpdateSince` metodu, metot her çağrıldığında yeni bir "teklif" oluşturacak şekilde geçersiz kılınır. Teklif, `createQuote` fonksiyonunu kullanarak oluşturulur ve teklif sabitine geçirilir. Metot, teklifin değerini ve güncellenmiş sayıyı içeren yeni bir kayıt döndürür.

Son olarak, yeni bir bildirici objesi oluşturulur ve `specificBaseNotifier`'ın metotları ve özellikleri ile birlikte döndürülür.

```js
makeQuoteNotifier(amountIn, brandOut) {
      AmountMath.coerce(actualBrandIn, amountIn);
      assertBrands(amountIn.brand, brandOut);

      // Wrap our underlying notifier with specific quotes.
      const specificBaseNotifier = harden({
        async getUpdateSince(updateCount = undefined) {
          // We use the same updateCount as our underlying notifier.
          const record = await E(notifier).getUpdateSince(updateCount);

          // We create a quote inline.
          const quote = createQuote(calcAmountOut => ({
            amountIn,
            amountOut: calcAmountOut(amountIn),
          }));
          assert(quote);

          const value = await quote;
          return harden({
            value,
            updateCount: record.updateCount,
          });
        },
      });

      /** @type {Notifier<PriceQuote>} */
      const specificNotifier = Far('QuoteNotifier', {
        ...makeNotifier(specificBaseNotifier),
        // TODO stop exposing baseNotifier methods directly.
        ...specificBaseNotifier,
      });
      return specificNotifier;
    }
```

Aşağıdaki test, [scaled price authority](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/test/unitTests/contracts/test-scaledPriceAuthority.js#L44), makeQuoteNotifier metodunun nasıl kullanılacağını göstermektedir.
MakeQuoteNotifier tarafından döndürülen bildirici nesnesinin getUpdateSince metodunu çağırarak, güncellenmiş teklif kaydı ve güncellenmiş sayı ile bir promise döndürecektir.

test('scaled price authority', /** @param {ExecutionContext} t */ async t => {

    ...
  const pa = await E(scaledPrice.publicFacet).getPriceAuthority();

  const notifier = E(pa).makeQuoteNotifier(
    AmountMath.make(t.context.ibcAtom.brand, 10n ** 6n),
    t.context.run.brand,
  );

  const {
    value: { quoteAmount: qa1 },
    updateCount: uc1,
  } = await E(notifier).getUpdateSince();
  t.deepEqual(qa1.value, [
    {
      amountIn: { brand: t.context.ibcAtom.brand, value: 10n ** 6n },
      amountOut: { brand: t.context.run.brand, value: 35_610_000n },
      timer,
      timestamp: 0n,
    },
  ]);

  ...

});


### quoteWanted

[quoteWanted](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/src/contractSupport/priceAuthority.js#L289) fonksiyonu, verilen bir `brandIn` ve `amountOut` için fiyat teklifi hesaplar.
Bu fonksiyon, yukarıdaki fonksiyonla benzerdir, tek fark, createQuote fonksiyonunun iki parametre `calcAmountOut` ve `calcAmountIn` almasıdır. Geri çağırma fonksiyonu, minimum bir amountOut garantileyen bir amountIn değerini belirlemek için calcAmountIn'i kullanır ve ardından verilen amountIn için gerçek amountOut'u hesaplamak için calcAmountOut'u kullanır.
Fonksiyon, hem amountIn hem de amountOut ile fiyat teklifini döndürür.

```js
    async quoteWanted(brandIn, amountOut) {
      AmountMath.coerce(actualBrandOut, amountOut);
      assertBrands(brandIn, amountOut.brand);

      await E(notifier).getUpdateSince();
      const quote = createQuote((calcAmountOut, calcAmountIn) => {
        // En az amountOut garantileyen bir amountIn belirlememiz gerekiyor.
        const amountIn = calcAmountIn(amountOut);
        const actualAmountOut = calcAmountOut(amountIn);
        AmountMath.isGTE(actualAmountOut, amountOut) ||
          assert.fail(
            X`Hesaplama olan ${actualAmountOut}, beklenen ${amountOut}'u karşılamadı`,
          );
        return { amountIn, amountOut };
      });
      assert(quote);
      return quote;
    },
```

Aşağıdaki test, [priceAuthority quoteWanted](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/test/unitTests/test-fakePriceAuthority.js#L72), quoteWanted metodunu nasıl kullanılacağını gösterir.
BrandIn olarak _moolaBrand_ ve amountOut olarak _bucks(400n)_ verilerek beklenen teklif döndürülür.
Bu örnekte, teklif miktarı `quote.quoteAmount.value[0]` özelliği kullanılarak erişilir.

```js
test("priceAuthority quoteWanted", async (t) => {
  const { moola, bucks, brands } = setup();
  const moolaBrand = brands.get("moola");
  assert(moolaBrand);
  const manualTimer = buildManualTimer(t.log, 0n, { eventLoopIteration });
  const priceAuthority = await makeTestPriceAuthority(
    brands,
    [20, 55],
    manualTimer
  );

  await E(manualTimer).tick();
  const quote = await E(priceAuthority).quoteWanted(moolaBrand, bucks(400n));
  const quoteAmount = quote.quoteAmount.value[0];
  t.is(1n, quoteAmount.timestamp);
  assertAmountsEqual(t, bucks(400n), quoteAmount.amountOut);
  assertAmountsEqual(t, moola(20n), quoteAmount.amountIn);
});
```

## Promise Fiyat Teklifi

`priceAuthority`'nin belirli koşulların, geliştirici tarafından belirlenen, gerçekleştiğinde çözülecek bir `promise` döndüren birkaç [metodu](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/src/contractSupport/priceAuthority.js#L339) vardır.
Bu metodlar:

```js
    quoteWhenLT: makeQuoteWhenOut(isLT),
    quoteWhenLTE: makeQuoteWhenOut(isLTE),
    quoteWhenGTE: makeQuoteWhenOut(isGTE),
    quoteWhenGT: makeQuoteWhenOut(isGT),
    mutableQuoteWhenLT: makeMutableQuote(isLT),
    mutableQuoteWhenLTE: makeMutableQuote(isLTE),
    mutableQuoteWhenGT: makeMutableQuote(isGT),
    mutableQuoteWhenGTE: makeMutableQuote(isGTE),
```

Yukarıdaki metodlar tarafından beklenen parametreler, `compareAmountsFn` olarak adlandırılan, AmountMath kütüphanesi kullanılarak oluşturulur. İlgili promise için koşulu bu parametre belirler.

```js
/**
 * @callback CompareAmount
 * @param {Amount} amount
 * @param {Amount} amountLimit
 * @returns {boolean}
 */

/** @type {CompareAmount} */
const isLT = (amount, amountLimit) => !AmountMath.isGTE(amount, amountLimit);

/** @type {CompareAmount} */
const isLTE = (amount, amountLimit) => AmountMath.isGTE(amountLimit, amount);

/** @type {CompareAmount} */
const isGTE = (amount, amountLimit) => AmountMath.isGTE(amount, amountLimit);

/** @type {CompareAmount} */
const isGT = (amount, amountLimit) => !AmountMath.isGTE(amountLimit, amount);
```

### quoteWhen

[makeQuoteWhenOut](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/src/contractSupport/priceAuthority.js#L85) fonksiyonu bir `compareAmountsFn` alır ve bir iç fonksiyon olan `quoteWhenOutTrigger`'ı döndürür. İç fonksiyon, `amountIn` ve `amountOutLimit` olmak üzere iki argüman alan asenkron bir fonksiyondur.

Fonksiyon, `makePromiseKit` kullanarak bir Promise objesi `triggerPK` oluşturur. Ardından `createInstantQuote` alarak bir `trigger` adlı asenkron bir fonksiyon oluşturur. Bu fonksiyon, createInstantQuote ve calcAmountOut'a dayanarak bir teklif oluşturmaya çalışır:

- Eğer tetikleyiciler seti tetikleyiciyi içermiyorsa, tetikleyici zaten ateşlendiği için hemen undefined döndürür.
- Eğer compareAmountsFn, calcAmountOut(amountIn) ve amountOutLimit geçildiğinde false dönerse, tetikleyici henüz ateşlememeli çünkü undefined döner.
- Eğer compareAmountsFn true dönerse, fonksiyon, amountIn ve amountOut özelliklerine sahip bir obje döndürerek bir teklif oluştur.

- Eğer compareAmountsFn doğru dönerse, işlev, amountIn ve amountOut özelliklerine sahip bir nesne döndürerek bir teklif oluşturur.
- Eğer createInstantQuote'nun döndürdüğü değer null ise, undefined döndürür.
- Eğer try bloğunda bir istisna atılırsa, tetikleyici vaadi reddedilir ve tetikleyici, tetikleyiciler kümesinden silinir.

Sonunda, tetikleyici tetikleyiciler kümesine eklenir ve tetikleyici fonksiyonu, argümanı olarak createQuote ile hemen çalıştırılır. İşlev daha sonra Promise nesnesi olan triggerPK'yi döndürecektir.

Aşağıdaki test, [priceAuthority quoteWhenLT](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/test/unitTests/test-fakePriceAuthority.js#L146), `quoteWhenLT` metodunun nasıl kullanılacağını gösterir.
AmountIn (moola(1n)) ve amountOutLimit (bucks(30n)) sağlayarak, bir PriceQuote vaadi döndürecektir.
Zaman, koşul tetiklenene kadar ileriye taşınacaktır (await E(manualTimer).tick()).

```js
test("priceAuthority quoteWhenLT", async (t) => {
  const { moola, bucks, brands } = setup();
  const manualTimer = buildManualTimer(t.log, 0n, { eventLoopIteration });
  const priceAuthority = await makeTestPriceAuthority(
    brands,
    [40, 30, 29],
    manualTimer
  );

  E(priceAuthority)
    .quoteWhenLT(moola(1n), bucks(30n))
    .then((quote) => {
      const quoteInAmount = quote.quoteAmount.value[0];
      // @ts-expect-error could be TimestampRecord
      t.is(3n, manualTimer.getCurrentTimestamp());
      t.is(3n, quoteInAmount.timestamp);
      assertAmountsEqual(t, bucks(29n), quoteInAmount.amountOut);
      assertAmountsEqual(t, moola(1n), quoteInAmount.amountIn);
    });

  await E(manualTimer).tick();
  await E(manualTimer).tick();
  await E(manualTimer).tick();
  await E(manualTimer).tick();
});
```

### mutableQuoteWhen

[makeMutableQuote](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/src/contractSupport/priceAuthority.js#L146) fonksiyonu, yukarıdaki işlevle çok benzer, ancak çağrıldığında, `mutableQuoteWhenOutTrigger` adında yeni bir işlev döndürür, bu da bir `mutableQuote` nesnesi döndürür.

MutableQuote nesnesi, üç metodla oluşturulur: `cancel`, `updateLevel`, ve `getPromise`:

- Cancel metodunu, mutableQuote ile ilişkilendirilmiş vaadi reddetmek için kullanılır.
- UpdateLevel metodunu, amountIn ve amountOutLimit değerlerini değiştirmek ve tetikleyiciyi yeniden ateşlemek için kullanılır.
- GetPromise metodunu, mutableQuote ile ilişkilendirilmiş vaadi döndürmek için kullanılır.

Aşağıdaki test, [temutableQuoteWhenLT: brands in/out matchst](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/test/unitTests/contracts/test-scaledPriceAuthority.js#L178), `mutableQuoteWhenLT` metodunun nasıl kullanılacağını gösterir.
AmountIn'i, _AmountMath.make(t.context.ibcAtom.brand, 10n \*\* 6n)_, ve amountOutLimit'i, _AmountMath.make(t.context.ibcAtom.brand, 10n \*\* 6n)_, sağlayarak, bir MutableQuote vaadi döndürecektir.
Zaman, _await E(timer).tick()_ kullanılarak ileriye taşınacak, bu da koşulun tetiklenmesine izin verecektir.

```js
test("mutableQuoteWhenLT: markalar giriş/çıkışta eşleşiyor", /** @param {ExecutionContext} t */ async (t) => {
  const timer = buildManualTimer(t.log);
  const makeSourcePrice = (valueIn, valueOut) =>
    makeRatio(valueOut, t.context.usdBrand, valueIn, t.context.atomBrand);
  const sourcePriceAuthority = makeManualPriceAuthority({
    actualBrandIn: t.context.atomBrand,
    actualBrandOut: t.context.usdBrand,
    initialPrice: makeSourcePrice(10n ** 5n, 35_6100n),
    timer,
  });

  const scaledPrice = await E(t.context.zoe).startInstance(
    t.context.scaledPriceInstallation,
    undefined,
    {
      sourcePriceAuthority,
      scaleIn: makeRatio(
        10n ** 5n,
        t.context.atomBrand,
        10n ** 6n,
        t.context.ibcAtom.brand
      ),
      scaleOut: makeRatio(
        10n ** 4n,
        t.context.usdBrand,
        10n ** 6n,
        t.context.run.brand
      ),
    }
  );

  const pa = await E(scaledPrice.publicFacet).getPriceAuthority();

  const mutableQuote = E(pa).mutableQuoteWhenLT(
    AmountMath.make(t.context.ibcAtom.brand, 10n ** 6n),
    AmountMath.make(t.context.run.brand, 32_430_100n)
  );
  const sourceNotifier = E(sourcePriceAuthority).makeQuoteNotifier(
    AmountMath.make(t.context.atomBrand, 10n ** 5n),
    t.context.usdBrand
  );

  const {
    value: { quoteAmount: sqa1 },
    updateCount: suc1,
  } = await E(sourceNotifier).getUpdateSince();
  t.deepEqual(sqa1.value, [
    {
      amountIn: { brand: t.context.atomBrand, value: 10n ** 5n },
      amountOut: { brand: t.context.usdBrand, value: 35_6100n },
      timer,
      timestamp: 0n,
    },
  ]);

  await E(timer).tick();
  sourcePriceAuthority.setPrice(makeSourcePrice(10n ** 5n, 30_4301n));
  const { quoteAmount: qa2 } = await E(mutableQuote).getPromise();
  t.deepEqual(qa2.value, [
    {
      amountIn: { brand: t.context.ibcAtom.brand, value: 1_000_000n },
      amountOut: { brand: t.context.run.brand, value: 30_430_100n },
      timer,
      timestamp: 1n,
    },
  ]);

  // kaynak fiyat teklifini kontrol et
  const {
    value: { quoteAmount: sqa2 },
  } = await E(sourceNotifier).getUpdateSince(suc1);
  t.deepEqual(sqa2.value, [
    {
      amountIn: { brand: t.context.atomBrand, value: 1_00000n },
      amountOut: { brand: t.context.usdBrand, value: 30_4301n },
      timer,
      timestamp: 1n,
    },
  ]);
});
```

Diğer örnekler için aşağıdaki testlere göz atmanızı öneririz:

- [güncelleme ile mutableQuoteWhen](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/test/unitTests/contracts/test-priceAggregator.js#L946)
- [mutableQuoteWhen'i iptal et](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/test/unitTests/contracts/test-priceAggregator.js#L1020)

`makeQuoteWhenOut` ve `makeMutableQuote` işlevleri arasındaki ana fark, `makeQuoteWhenOut`'un belirli bir koşul yerine getirildiğinde bir teklifle çözülen bir söz veren `quoteWhenOutTrigger` işlevi oluşturmasıdır, buna karşın `makeMutableQuote` bir `mutableQuoteWhenOutTrigger` işlevi oluşturur ki bu da güncellenebilen ve iptal edilebilen bir mutable teklif verir.

## Manuel Fiyat Otoritesi

[makeManualPriceAuthority](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/tools/manualPriceAuthority.js) işlevi, `PriceAuthority` arabirimini uygulayan ve geri dönen bir nesne oluşturan ve döndüren bir manuel fiyat otoritesini temsil eden bir nesne döndürür, aşağıdaki ek yöntemle:

- setPrice: talep edilen varlık oranının mevcut fiyatını değiştirmeyi sağlayan bir işlev.

`createQuote` işlevi, mevcut fiyat için bir fiyat teklifi oluşturur. Teklif, bir teklifi alarak `quoteMint` kullanarak ödemeye dönüştüren ve `quote payment` ve `quote amount` döndüren `authenticateQuote` işlevi kullanılarak doğrulanır. `calcAmountOut` ve `calcAmountIn` işlevleri, mevcut fiyata dayalı olarak çıktı ve giriş miktarlarını hesaplar.

Son olarak, timer, createQuote işlevi, giriş ve çıkış markaları, teklif veren ve bildiriciyi içeren `priceAuthorityOptions` nesnesini alan `makeOnewayPriceAuthorityKit` yardımcı programı kullanılarak bir priceAuthority nesnesi oluşturulur.
PriceAuthority, setPrice yöntemiyle birlikte döndürülür.

```js
/**
 *
 * @param {object} options
 * @param {Brand} options.actualBrandIn
 * @param {Brand} options.actualBrandOut
 * @param {Ratio} options.initialPrice
 * @param {TimerService} options.timer
 * @param {IssuerKit<'set'>} [options.quoteIssuerKit]
 * @returns {PriceAuthority & { setPrice: (Ratio) => void }}
 */
export function makeManualPriceAuthority(options) {

  const {
    actualBrandIn,
    actualBrandOut,
    initialPrice, // brandOut / brandIn
    timer,
    quoteIssuerKit = makeIssuerKit('quote', AssetKind.SET),
  } = options;

    ...

  function createQuote(priceQuery) {
    const quote = priceQuery(calcAmountOut, calcAmountIn);
    if (!quote)
      return undefined;
    }

    const { amountIn, amountOut } = quote;
    return E(timer)
      .getCurrentTimestamp()
      .then(now =>
        authenticateQuote([{ amountIn, amountOut, timer, timestamp: now }]),
      );
  }

  /* --* @type {ERef<Notifier<Timestamp>>} */
  const priceAuthorityOptions = harden({
    timer,
    createQuote,
    actualBrandIn,
    actualBrandOut,
    quoteIssuer,
    notifier,
  });

  const {
    priceAuthority,
    adminFacet: { fireTriggers },
  } = makeOnewayPriceAuthorityKit(priceAuthorityOptions);

  return Far('ManualPriceAuthority', {
    setPrice: newPrice => {
      currentPrice = newPrice;
      updater.updateState(currentPrice);
      fireTriggers(createQuote);
    },
    ...priceAuthority,
  });
}
```

Aşağıdaki [ölçeklendirilmiş fiyat yetkilisi](https://github.com/Agoric/agoric-sdk/blob/b36fcb2d832ec154019cb6ae0a5dda17f94207eb/packages/zoe/test/unitTests/contracts/test-scaledPriceAuthority.js#L83) testi, nasıl yapılacağını gösterir "makeManualPriceAuthority" işlevini kullanın. makeManualPriceAuthority(), makeQuoteNotifier yöntemini kullanarak quoteNotifier'ı almak ve setPrice yöntemini kullanarak fiyatı güncellemek için kullanılan bir manualPriceAuthority, "sourcePriceAuthority" döndürür.

```js
test("scaled price authority", /** @param {ExecutionContext} t */ async (t) => {
  const timer = buildManualTimer(t.log);
  const makeSourcePrice = (valueIn, valueOut) =>
    makeRatio(valueOut, t.context.usdBrand, valueIn, t.context.atomBrand);
  const sourcePriceAuthority = makeManualPriceAuthority({
    actualBrandIn: t.context.atomBrand,
    actualBrandOut: t.context.usdBrand,
    initialPrice: makeSourcePrice(10n ** 5n, 35_6100n),
    timer,
  });

  const scaledPrice = await E(t.context.zoe).startInstance(
    t.context.scaledPriceInstallation,
    undefined,
    {
      sourcePriceAuthority,
      scaleIn: makeRatio(
        10n ** 5n,
        t.context.atomBrand,
        10n ** 6n,
        t.context.ibcAtom.brand
      ),
      scaleOut: makeRatio(
        10n ** 4n,
        t.context.usdBrand,
        10n ** 6n,
        t.context.run.brand
      ),
    }
  );

  const pa = await E(scaledPrice.publicFacet).getPriceAuthority();

  const notifier = E(pa).makeQuoteNotifier(
    AmountMath.make(t.context.ibcAtom.brand, 10n ** 6n),
    t.context.run.brand
  );
  const sourceNotifier = E(sourcePriceAuthority).makeQuoteNotifier(
    AmountMath.make(t.context.atomBrand, 10n ** 5n),
    t.context.usdBrand
  );

  const {
    value: { quoteAmount: qa1 },
    updateCount: uc1,
  } = await E(notifier).getUpdateSince();
  t.deepEqual(qa1.value, [
    {
      amountIn: { brand: t.context.ibcAtom.brand, value: 10n ** 6n },
      amountOut: { brand: t.context.run.brand, value: 35_610_000n },
      timer,
      timestamp: 0n,
    },
  ]);

  const {
    value: { quoteAmount: sqa1 },
    updateCount: suc1,
  } = await E(sourceNotifier).getUpdateSince();
  t.deepEqual(sqa1.value, [
    {
      amountIn: { brand: t.context.atomBrand, value: 10n ** 5n },
      amountOut: { brand: t.context.usdBrand, value: 35_6100n },
      timer,
      timestamp: 0n,
    },
  ]);

  await E(timer).tick();
  sourcePriceAuthority.setPrice(makeSourcePrice(10n ** 5n, 32_4301n));
  const {
    value: { quoteAmount: qa2 },
  } = await E(notifier).getUpdateSince(uc1);
  t.deepEqual(qa2.value, [
    {
      amountIn: { brand: t.context.ibcAtom.brand, value: 1_000_000n },
      amountOut: { brand: t.context.run.brand, value: 32_430_100n },
      timer,
      timestamp: 1n,
    },
  ]);

  const {
    value: { quoteAmount: sqa2 },
  } = await E(sourceNotifier).getUpdateSince(suc1);
  t.deepEqual(sqa2.value, [
    {
      amountIn: { brand: t.context.atomBrand, value: 1_00000n },
      amountOut: { brand: t.context.usdBrand, value: 32_4301n },
      timer,
      timestamp: 1n,
    },
  ]);
});
```
