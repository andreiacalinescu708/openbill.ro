# Plan Implementare Discount Secvențial

## Obiectiv
Implementare sistem discount stack-based în aplicația openBill:
- User adaugă produse, apoi apasă "+ Linie Discount"
- Discount se aplică produselor de DEASUPRA liniei
- Se pot adăuga multiple discounturi secvențial
- Max discount 20%
- Ștergere discount cu recalculare

## 1. Backend (server.cjs)

### 1.1 Endpoint nou: POST /api/orders/:id/apply-discount
```javascript
// Body: { percent: 10 }
// Calculează suma produselor deasupra ultimei linii discount
// Adaugă item nou: { type: "discount", percent, amount, baseAmount }
```

### 1.2 Endpoint nou: DELETE /api/orders/:id/discount/:index
```javascript
// Șterge linia discount la indexul specificat
// Nu recalculează nimic - produsele de deasupra rămân fără discount
```

### 1.3 Modificare GET /api/orders/:id
```javascript
// Returnează items cu type inclus
```

### 1.4 Modificare sendDraftToSmartBill()
```javascript
// Mapează items cu type="discount" către SmartBill format:
// { name: "Discount X%", price: -amount, isDiscount: true, ... }
```

## 2. Frontend (comanda.html / script.js)

### 2.1 UI Coș
- Afișare items cu `type` diferit vizual
- Discounturi: badge galben, X mic de ștergere
- Suma discountului calculată live

### 2.2 Buton "+ Adaugă discount"
- Dialog/modal: input procent (1-20)
- Validare: să existe produse deasupra
- API call și refresh listă

### 2.3 Buton Ștergere Discount (X)
- Confirmare? Nu, direct ștergere
- API call DELETE și refresh

### 2.4 Calcul Total
```
Total = sum(products) + sum(discounts)
```

## 3. DB Schema

Fără modificări schema - folosim `items` JSONB existent cu câmp nou `type`.

## 4. SmartBill Integration

```javascript
// Discount line in SmartBill payload:
{
  name: "Discount 10%",
  quantity: 1,
  price: -6.80,  // negativ!
  isDiscount: true,
  isService: false,
  taxPercentage: 21,  // aceeași TVA ca produsele
  ...
}
```

## 5. Test Cases

1. Adaugă 2 produse → Discount 10% → Verifică sumă corectă
2. Adaugă produs după discount → Nu se aplică discount
3. Șterge discount → Produsele rămân fără discount
4. Trimite la SmartBill → Verifică format corect
5. Discount 25% → Eroare validare
6. Discount fără produse deasupra → Eroare validare

## 6. Secvență Implementare

1. ✅ Backend: GET orders cu type
2. ✅ Backend: POST apply-discount endpoint
3. ✅ Backend: DELETE discount endpoint
4. ✅ Backend: SmartBill payload cu discount
5. ✅ Frontend: Afișare items cu type
6. ✅ Frontend: Buton + modal discount
7. ✅ Frontend: Buton ștergere discount
8. ✅ Frontend: Calcul total corect
9. ✅ Test end-to-end

## Note
- Multi-tenant: toate query-urile folosesc `${schemaName}`
- Backwards compat: items fără type sunt tratate ca "product"
- Validare: max 20% discount, min 1%
