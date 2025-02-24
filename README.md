
# Naloga - Verde 365

## Naloga: Sistem za sledenje porabi AI žetonov/kreditov

Oblikujte sistem za sledenje porabi žetonov/kreditov pri uporabi različnih modelov umetne inteligence. Potrebujemo način, da shranjujemo podatke o porabi, da lahko strankam zaračunamo po dejanski uporabi. Idealno bi bilo, da imamo informacije o porabljenih žetonih in njihovi ceni. Te podatke bi beležili ob vsakem klicu AI storitve (te storitve ni treba implementirati, lahko si izmislite njen API).

### Predpostavke

- Uporabljali bomo več različnih AI modelov, različnih ponudnikov
- Cene modelov so lahko fiksne (vnesene ročno) in se spreminjajo ročno, ko se cena modela spremeni. Spremembe cen so običajno navzdol.
- Lahko predpostavimo, da pri vsakem API klicu pridobimo podatke o vhodnih/izhodnih žetonih.

### Zahteve

1. **Podatkovni model (shema baze):** Oblikujte shemo baze podatkov, ki bo shranjevala potrebne informacije. Navedite vse potrebne atribute in njihove tipe podatkov. Razložite razmerja med tabelami v kolikor so potrebna. Lahko uporabite katerikoli diagramski ali risalni pripomoček.
2. **Programski vmesnik:** Opišite (lahko tudi z uporabo psevdokode) kako bi izgledal API (vmesnik) za beleženje porabe žetonov. Katere podatke bi morali poslati API-ju ob vsakem klicu? Kakšen bi bil odziv API-ja?
3. **Primer izračuna stroškov:** Pokažite primer izračuna stroškov za določeno uporabo AI modela na podlagi shranjenih podatkov.

**Predpostavke in razširitve:** Navedite vse dodatne predpostavke, ki ste jih naredili pri oblikovanju sistema. Razmislite tudi o možnih razširitvah sistema v prihodnosti

## Rešitev

### Shema podatkovne baze

![diagram podatkovne baze](https://i.ibb.co/N2Z4pf2Q/Untitled-2.png)

```sql
// MySQL schema

CREATE TABLE `users` (
  `id` integer PRIMARY KEY,
  `username` varchar(255),
  `created_at` timestamp
);

CREATE TABLE `tokens` (
  `id` integer PRIMARY KEY,
  `user_id` integer,
  `provider_id` integer,
  `tokens_used` integer,
  `tokens_received` integer
);

CREATE TABLE `providers` (
  `id` integer PRIMARY KEY,
  `name` varchar(255),
  `token_price` float
);

ALTER TABLE `tokens` ADD FOREIGN KEY (`user_id`) REFERENCES `users` (`id`);
ALTER TABLE `tokens` ADD FOREIGN KEY (`provider_id`) REFERENCES `providers` (`id`);
```

### Primer API klica

Za primer privzamemo, da ima uporabnik "aljaz" ID 1 in AI storitev "chat_gpt" ID 2, status žetonov pa ima ID 3.

```ts
fetch("verde365assignment.com/api/user/1/prompt", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        provider: "chat_gpt"
    })
})
.then(response => response.json());
```

odgovor API-ja:

```json
{
     "provider_id": "2",
     "tokens_used": "1"
}
```

### Izračun porabe

```ts
fetch("verde365assignment.com/api/user/1/get_tokens", {
    method: "GET",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        provider_id: 2
    })
}).then(response => response.json());
```

API odgovor:

```json
{
    "id": 3,
    "user_id": 1,
    "provider_id": 2,
    "tokens_used": 34
}
```

Če privzamemo, da smo pred tem pridobili tudi ceno žetona za ponudnika, ki znaša 0.15 EUR na žeton, je izračun sledeč:

```ts
const user = user_response;
const tokens = tokens_response;
const provider = provider_response;

let price = tokens_response.tokens_used * provider.token_price;

console.log(`price: ${tokens_response.tokens_used} * ${provider.token_price} = ${price});
```

Output:

```console
"price: 34 * 0.15 = 5.10"
```

### Razširitve

Ker je to le koncept, nisem upošteval, da obstaj možnost, da se obračun naredi za določen časovni interval, za katerega se je stranka odločila ob registraciji na storitev.
