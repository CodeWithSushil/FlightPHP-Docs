# Konfigurācija

## Pārskats 

Flight nodrošina vienkāršu veidu, kā konfigurēt dažādus ietvara aspektus, lai tie atbilstu jūsu lietojumprogrammas vajadzībām. Dažas ir iestatītas pēc noklusējuma, bet jūs varat tās ignorēt pēc vajadzības. Jūs varat arī iestatīt savus mainīgos, lai tos izmantotu visā lietojumprogrammā.

## Izpratne

Jūs varat pielāgot noteiktas Flight uzvedības, iestatot konfigurācijas vērtības
izmantojot `set` metodi.

```php
Flight::set('flight.log_errors', true);
```

Failā `app/config/config.php` jūs varat redzēt visus noklusējuma konfigurācijas mainīgos, kas jums ir pieejami.

## Pamata lietošana

### Flight konfigurācijas opcijas

Tālāk ir saraksts ar visām pieejamajām konfigurācijas iestatījumiem:

- **flight.base_url** `?string` - Ignorējiet pieprasījuma bāzes URL, ja Flight darbojas apakšdirektorijā. (noklusējums: null)
- **flight.case_sensitive** `bool` - Lielo un mazo burtu jutīga atbilstība URL. (noklusējums: false)
- **flight.handle_errors** `bool` - Ļauj Flight apstrādāt visas kļūdas iekšēji. (noklusējums: true)
  - Ja vēlaties, lai Flight apstrādā kļūdas tā vietā, lai izmantotu noklusējuma PHP uzvedību, tam jābūt true.
  - Ja jums ir instalēts [Tracy](/awesome-plugins/tracy), jūs vēlaties iestatīt to uz false, lai Tracy varētu apstrādāt kļūdas.
  - Ja jums ir instalēts [APM](/awesome-plugins/apm) spraudnis, jūs vēlaties iestatīt to uz true, lai APM varētu reģistrēt kļūdas.
- **flight.log_errors** `bool` - Reģistrēt kļūdas tīmekļa servera kļūdu žurnāla failā. (noklusējums: false)
  - Ja jums ir instalēts [Tracy](/awesome-plugins/tracy), Tracy reģistrēs kļūdas, pamatojoties uz Tracy konfigurācijām, nevis šo konfigurāciju.
- **flight.debug** `bool` - Izvadīt detalizētu kļūdu informāciju (izņēmuma ziņojumu, kodu un steka izsekošanu) pārlūkprogrammā, kad rodas kļūda. (noklusējums: false)
  - **Nekad neieslēdziet to ražošanā** — tas nopludina iekšējās lietojumprogrammas detaļas. Izmantojiet to tikai lokālai izstrādei vai testēšanai.
  - Kad `false`, tā vietā tiek parādīta vispārīga `500 Internal Server Error`. Apvienojiet ar `flight.log_errors`, lai tvertu kļūdas servera pusē.
- **flight.allow_method_override** `bool` - Ļauj ignorēt HTTP metodi, izmantojot `X-HTTP-Method-Override` pieprasījuma galveni vai `_method` lauku POST ķermenī. (noklusējums: true)
  - **Ieteicams iestatīt uz `false`** lietojumprogrammām, kurām nav nepieciešama uz HTML formas balstīta metodes viltošana, jo tas novērš klientiem iespēju viltot `DELETE` vai `PUT` pieprasījumus, izmantojot standarta POST formu.
  - Skatiet [Drošība](/learn/security#flight-configuration-hardening) lai iegūtu vairāk detaļu.
- **flight.views.path** `string` - Direktorijs, kas satur skata veidņu failus. (noklusējums: ./views)
- **flight.views.extension** `string` - Skata veidņu faila paplašinājums. (noklusējums: .php)
- **flight.content_length** `bool` - Iestatīt `Content-Length` galveni. (noklusējums: true)
  - Ja izmantojat [Tracy](/awesome-plugins/tracy), tas ir jāiestata uz false, lai Tracy varētu pareizi renderēt.
- **flight.v2.output_buffering** `bool` - Izmantot mantoto izvades buferizāciju. Skatiet [migrācija uz v3](migrating-to-v3). (noklusējums: false)

### Ielādētāja konfigurācija

Papildus ir vēl viens konfigurācijas iestatījums ielādētājam. Tas ļaus jums 
automātiski ielādēt klases ar `_` klases nosaukumā.

```php
// Enable class loading with underscores
// Defaulted to true
Loader::$v2ClassLoading = false;
```

### Mainīgie

Flight ļauj saglabāt mainīgos, lai tos varētu izmantot jebkurā vietā jūsu lietojumprogrammā.

```php
// Save your variable
Flight::set('id', 123);

// Elsewhere in your application
$id = Flight::get('id');
```
Lai pārbaudītu, vai mainīgais ir iestatīts, varat darīt:

```php
if (Flight::has('id')) {
  // Do something
}
```

Jūs varat notīrīt mainīgo, darot:

```php
// Clears the id variable
Flight::clear('id');

// Clears all variables
Flight::clear();
```

> **Piezīme:** Tikai tāpēc, ka jūs varat iestatīt mainīgo, nenozīmē, ka jums vajadzētu to darīt. Izmantojiet šo funkciju taupīgi. Iemesls ir tāds, ka viss, kas šeit tiek saglabāts, kļūst par globālo mainīgo. Globālie mainīgie ir slikti, jo tos var mainīt no jebkuras vietas jūsu lietojumprogrammā, padarot grūti atrast kļūdas. Turklāt tas var sarežģīt tādas lietas kā [vienības testēšana](/guides/unit-testing).

### Kļūdas un izņēmumi

Visas kļūdas un izņēmumi tiek Flight notverti un nodoti `error` metodei.
ja `flight.handle_errors` ir iestatīts uz true.

Noklusējuma uzvedība ir nosūtīt vispārēju `HTTP 500 Internal Server Error`
atbildi ar dažiem kļūdu informācijas datiem.

Jūs varat [ignorēt](/learn/extending) šo uzvedību savām vajadzībām:

```php
Flight::map('error', function (Throwable $error) {
  // Handle error
  echo $error->getTraceAsString();
});
```

Pēc noklusējuma kļūdas netiek reģistrētas tīmekļa serverī. Jūs varat to iespējot,
mainot konfigurāciju:

```php
Flight::set('flight.log_errors', true);
```

#### 404 Not Found

Kad URL nevar atrast, Flight izsauc `notFound` metodi. Noklusējuma
uzvedība ir nosūtīt `HTTP 404 Not Found` atbildi ar vienkāršu ziņojumu.

Jūs varat [ignorēt](/learn/extending) šo uzvedību savām vajadzībām:

```php
Flight::map('notFound', function () {
  // Handle not found
});
```

## Skatiet arī
- [Flight paplašināšana](/learn/extending) - Kā paplašināt un pielāgot Flight galveno funkcionalitāti.
- [Vienības testēšana](/guides/unit-testing) - Kā rakstīt vienības testus savai Flight lietojumprogrammai.
- [Tracy](/awesome-plugins/tracy) - Spraudnis uzlabotai kļūdu apstrādei un atkļūdošanai.
- [Tracy paplašinājumi](/awesome-plugins/tracy_extensions) - Paplašinājumi Tracy integrēšanai ar Flight.
- [APM](/awesome-plugins/apm) - Spraudnis lietojumprogrammas veiktspējas uzraudzībai un kļūdu izsekošanai.

## Problēmu novēršana
- Ja jums ir problēmas ar visu jūsu konfigurācijas vērtību atrašanu, varat darīt `var_dump(Flight::get());`

## Izmaiņu žurnāls
- v3.18.1 - Pievienotas `flight.debug` un `flight.allow_method_override` konfigurācijas opcijas.
- v3.5.0 - Pievienota konfigurācija `flight.v2.output_buffering`, lai atbalstītu mantoto izvades buferizācijas uzvedību.
- v2.0 - Pievienotas galvenās konfigurācijas.