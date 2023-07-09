# Ders Dört - Ödev

## Ödev Açıklaması:

1. Yeni bir sözleşme oluşturun;
2. Bazı NFT'ler basın;
3. Kullanıcının bazı basılmış NFT'leri Zoe sellItems sözleşmesi aracılığıyla satmasına izin veren bir fonksiyon oluşturun;
4. Yukarıdaki sözleşme için yeni bir test dosyası oluşturun;

## Sıralı diyagram

```mermaid
sequenceDiagram
    actor Alice
    participant Contract
    participant SellItems
    note over Alice,SellItems: NFT Sat
    Alice->>+Contract: sell( nftIDs, pricePerNFT, sellItemsInstalation)
    Contract->>+Contract: NFT'ler bas
    Contract->>+Contract: SellItems örneğini başlat
    Contract->>+SellItems: offer(invitation, proposal, ...)
    SellItems -->>Contract: sellItemsCreatorSeat
    Contract -->>Alice: creatorFacet, publicFacet, ...
    note over Alice,SellItems: NFT Satın Al
    Alice->>+SellItems: makeBuyerInvitation()
    SellItems -->>Alice: Invitation
    Alice->>+SellItems: offer(invitation, proposal, ...)
    SellItems -->>Alice: buyerSeat
```
