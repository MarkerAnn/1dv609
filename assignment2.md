# Översikt

**Projektets namn:** Budget **Projektägare**: range-of-motion **Syfte**: Hjälper
användare att hantera sina ekonomiska resurser genom att spåra inkomster,
utgifter och budgetar. **Programmeringsspråk**: PHP **Testramverk**: PHPUnit för
enhetstester, Laravel Dusk för funktionstester. **Typ av applikation**:
Webbapplikation med ett användargränssnitt och API

[https://github.com/range-of-motion/budget](https://github.com/range-of-motion/budget)

# Kravspecifikation

Det fanns ingen explicit kravspecifikation, men efter att ha granskat README då
kunde jag få ut följande:

#### Funktionella krav

**Användarhantering**

- Registrering och verifiering
- Inloggning och autentisering
- Lösenordshantering

**Transaktionshantering**

- Lägg till, redigera och ta bort transaktioner
- Organisera transaktioner med taggar

**Kvittohantering**

- Ladda upp och organisera kvitton

**Import av transaktioner**

- Importera transaktioner via CSV-filer

**Rapportering**

- Generera visualiserade rapporter:
  - Veckobalans
  - Mest kostsamma taggar

**Fleranvändarstöd**

- Stöd för flera valutor
- Tillgänglighet på flera språk

**Sammanfattningar**

- Veckosammanfattningar via e-post

#### Tekniska krav

**Docker-stöd**

- Möjlighet att köra applikationen med Docker och Docker Compose
- Inkluderar MySQL-databas

# Utvecklingsresa

Baserat på projektets migrationsfiler så kan man se följande när det kommer till
utvecklingen:

**Tidpunkt för start:**

- Projektets databasstruktur indikerar att det första schemat skapades i oktober
  2014, enligt migreringsfilen  
  `2014_10_11_000000_create_currencies_table.php`.

**Tidiga funktioner:**

- Hantering av användare, inklusive verifiering och autentisering  
  (`2014_10_12_000000_create_users_table.php`).
- Introduktion av tags och transaktioner  
  (`2017_07_19_000000_create_tags_table.php` och
  `2017_07_20_000000_create_spendings_table.php`).
- Implementering av återkommande betalningar och intäkter  
  (`2018_09_13_112448_create_recurrings_table.php`).

**Senaste migrationsfilen:**

- Här la man till Mexikanska pesos som currency i currency table  
  (`2024_08_31_211536_insert_mexican_peso_into_currencies_table.php`).

#### Fortsatt utveckling

Projektet har i skrivandets stund 18 issues och många av dessa är skrivna av
grundaren själv, vilket tyder på att det finns planer för vidare utveckling av
projektet.

Det planeras exempelvis för att uppgradera projektet till Laravel 11 samt att
byta designmönster från Repository Design Pattern till något som är mer lämpligt
när man använder Laravel.

# Testramverk och grundläggande struktur

Projektet använder PHPUnit som sitt primära testramverk, vilket är standard för
PHP-applikationer. Detta kan man se genom hela testkatalogen där testklasser
ärver från `TestCase`. Projektet använder också Laravel Dusk för
webbläsartestning, det är ett verktyg för end-to-end-testning. Laravel Dusk
använder en webbläsarsimulator (som Chromium eller ChromeDriver) för att testa
applikationens gränssnitt och användarflöde

Testerna är organiserade i tre huvudkategorier:

1. Unit Tests (finns i `tests/Unit`) 17st
2. Feature Tests (finns i `tests/Feature`) 12 st
3. Browser Tests (finns i `tests/Browser`) 3st

# Unit Tests

> [!NOTE] Enhetstester Dessa tester fokuserar på att testa enskilda komponenter,
> isolerade.

Ett exempel på detta finns i `tests/Unit/Models/SpaceTest.php`. Här testas
Space-modellen:

```php
public function testAbbreviatedNameAttribute(): void
{
    $space = Space::factory()
        ->create([
            'name' => 'Hello world',
        ]);

    $this->assertEquals('Hel...', $space->abbreviated_name);
}
```

Detta test visar flera viktiga punkter:

- Det testar en enda funktion (Single Responsibility)
- Det har ett tydligt förväntat resultat
- Det använder Laravel's factory-system för att skapa testdata

testet testar accessorn `abbreviatedName` i `Space`-modellen. En accessor i
Laravel används för att definiera hur ett attributs värde ska beräknas eller
modifieras när det hämtas. Här skapar `abbreviatedName` ett dynamiskt attribut
(`$space->abbreviated_name`) som inte lagras i databasen utan genereras vid
behov. `Str::limit` är en Laravel-hjälpmetod som förkortar en sträng till ett
visst antal tecken och lägger till `...` om strängen förkortas. I det här fall
begränsar `Str::limit($this->name, 3)` namnet till de första tre tecknen, följt
av `...` om strängen är längre än tre tecken. Så, om `$this->name` är
`"Hello world"`, blir `abbreviated_name` `"Hel..."`.

Ett annat exempel på enhetstestning finns i `tests/Unit/HelperTest.php` där
hjälpfunktioner testas:

```php
public function testRawNumberConversion(): void
{
    $cases = [
        [
            'rawNumber' => 9.70,
            'expected' => 970
        ],
        [
            'rawNumber' => 0.01,
            'expected' => 1
        ]
    ];


    foreach ($cases as $case) {
        $result = Helper::rawNumberToInteger($case['rawNumber']);
        $this->assertEquals($case['expected'], $result);
    }
}
```

Detta test visar användningen av datatablåer (data providers) för att testa
samma funktion med olika ingångsvärden.

Först skapar testet en array med flera testfall där:

- `rawNumber` är ett flyttal (decimaltal).
- `expected` är det förväntade resultatet efter konvertering till heltal.

För varje testfall sen:

- Anropar metoden `rawNumberToInteger` med `rawNumber`.
- Jämför det faktiska resultatet (`$result`) med det förväntade värdet
  (`$case['expected']`).

Metoden som testet testar ser utsåhär:

```php
public static function rawNumberToInteger(float $rawNumber): int
{
    return (int) round($rawNumber * 100);
}

```

**Multiplicerar med 100:** `$rawNumber * 100` konverterar ett flyttal till ett
nummer utan decimaler. Exempel: `9.70 * 100 = 970.0`.

**Avrundar värdet:** `round(...)` avrundar värdet till närmaste heltal. Exempel:
`970.499` blir `970`, och `970.5` blir `971`.

**Casta till `int`:** `(int)` omvandlar resultatet till ett heltal. Exempel:
`970.0` blir `970`.

**Testet tillsammans med metoden:**

Första testfallet:

- **Input:** `9.70`
- **Steg i metoden:**
  1. Multiplicera: `9.70 * 100 = 970.0`
  2. Avrunda: `round(970.0) = 970`
  3. Casta: `(int) 970 = 970`
- **Förväntat resultat:** `970`
- **Testresultat:** Passerar.

Andra testfallet:

- **Input:** `0.01`
- **Steg i metoden:**
  1. Multiplicera: `0.01 * 100 = 1.0`
  2. Avrunda: `round(1.0) = 1`
  3. Casta: `(int) 1 = 1`
- **Förväntat resultat:** `1`
- **Testresultat:** Passerar.

# Feature Tests - Integration och Funktionalitet

> [!NOTE] Feature tester Feature-testerna i projektet testar större
> funktionalitetsenheter och hur olika komponenter fungerar tillsammans

Ett bra exempel finns i `tests/Feature/CreateEarningTest.php`:

```php
public function testSuccessfulCreation(): void
{
    $response = $this
        ->followingRedirects()
        ->actingAs($this->user)
        ->withSession(['space_id' => $this->space->id])
        ->postJson('/earnings', [
            'date' => date('Y-m-d'),
            'description' => 'Something for the test',
            'amount' => '9.99'
        ]);

    $response
        ->assertStatus(200);
}
```

Detta test visar flera viktiga aspekter av feature-testning:

- Det testar hela flödet från HTTP-request till respons
- Det simulerar en autentiserad användare med `actingAs()`
- Det testar API-endpoints med JSON-data

Först **Simulerar en autentiserad användare:**

```php
    $this->actingAs($this->user)
```

- Metoden **`actingAs`** används för att logga in användaren (skapad i
  `setUp`-metoden i testklassen).
- Detta säkerställer att förfrågan sker från en auktoriserad användare.

**Skapar en session**

```php
->withSession(['space_id' => $this->space->id])
```

- En session-variabel `space_id` sätts till det aktuella `Space`-objektets ID.
- Detta kan användas av applikationen för att koppla förfrågan till ett
  specifikt "Space".

**Gör en POST-förfrågan**

```php
->postJson('/earnings', [
    'date' => date('Y-m-d'),
    'description' => 'Something for the test',
    'amount' => '9.99'
])
```

Skickar JSON-data till `/earnings`-endpointen med följande:

- **`date`**: Dagens datum i formatet `YYYY-MM-DD`.
- **`description`**: En kort beskrivning av inkomsten.
- **`amount`**: Beloppet för inkomsten, här satt till `'9.99'`.

**Kontrollerar HTTP-statud**

```php
$response->assertStatus(200);
```

Testar att svaret från servern är **200** (OK), vilket indikerar att resursen
skapades framgångsrikt.

# Browser Tests - End-to-End Testing

> [!NOTE] End-To-End testing Kontrollerar så att frontend och backend fungerar
> tillsammans

Projektet använder Laravel Dusk för automatiserad webbläsartestning. I
`tests/Browser/SmokeTest.php` finns ett grundläggande exempel:

```php
public function testBoot(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
                ->assertSee('Log in');
    });
}
```

Detta är ett så kallat "smoke test" som verifierar att applikationen åtminstone
startar och visar inloggningssidan.

**Vad gör testet:**

```php
$this->browse()
```

- Denna metod används för att starta en webbläsarsession i Laravel Dusk.
- Metoden tar en anonym funktion som en parameter, där vi kan definiera vad
  webbläsaren ska göra.

**Simulera en webbläsare**

```php
function (Browser $browser) { ... }
```

`$browser` är en instans av klassen `Browser` som hanterar
webbläsarinteraktioner, som att besöka URL:er, fylla i formulär, klicka på
knappar osv.

**Navigera till startsidan**

```php
$browser->visit('/')
```

- Metoden **`visit('/')`** navigerar till root-URL för applikationen,
  startsidan.
- `'/'` representerar root-path för applikationen.

**Kontrollerar att sidan innehåller "Log in"**

```php
->assertSee('Log in');
```

- **`assertSee`** verifierar att texten `"Log in"` är synlig på sidan som
  laddas.
- Testet misslyckas om:
  - Texten `"Log in"` inte finns på sidan.
  - Sidan inte laddas korrekt.

# Repository pattern och testning

Något jag inte hade stött på tidigare var `repository pattern`
Repository-mönstret är ett designmönster som används för att hantera
dataåtkomstlogik i applikationer. Det fungerar som ett abstraktionslager mellan
applikationens affärslogik och datakällan (t.ex. en databas). Med andra ord,
istället för att direkt interagera med databasfrågor (SQL) i modellerna eller
tjänster, kapslas all dataåtkomst in i ett repository.

> [!NOTE] Repository Testing Repository tester kontrollerar logiken som hanterar
> databasoperationer och affärslogik

I `tests/Unit/Repositories/BudgetRepositoryTest.php` ser vi ett omfattande
exempel:

```php
public function testGetSpentByIdUsingMonthlyBudget(): void
{
    $this->withSession(['space_id' => $this->spaceId]);

    $tag = Tag::factory()->create(['space_id' => $this->spaceId]);

    $budget = Budget::factory()->create([
        'space_id' => $this->spaceId,
        'tag_id' => $tag->id,
        'period' => 'monthly'
    ]);

    // Test utan utgifter
    $this->assertEquals(0, $this->budgetRepository->getSpentById($budget->id));

    // Lägg till utgifter och testa igen
    Spending::factory()->create([
        'space_id' => $this->spaceId,
        'tag_id' => $tag->id,
        'happened_on' => date('Y-m-d'),
        'amount' => 123
    ]);

    $this->assertEquals(123, $this->budgetRepository->getSpentById($budget->id));
}
```

**Förberedelse av testmiljö**

```php
$this->withSession(['space_id' => $this->spaceId]);
```

- Konfigurerar en aktiv session för testet
- Simulerar en inloggad användare med tillgång till ett specifikt space

**Skapar testdata med factories**

```php
$tag = Tag::factory()->create(['space_id' => $this->spaceId]);
$budget = Budget::factory()->create([...]);
```

- Använder Laravels factory-system för att skapa testdata
- Skapar en tag och en budget som är kopplade till det aktiva space:et
- Factory-metoden genererar och sparar data direkt i testdatabasen

**Första testet - Tom budget**

```php
$this->assertEquals(0, $this->budgetRepository->getSpentById($budget->id));
```

- Kontrollerar att en nyligen skapad budget inte har några utgifter
- Verifierar att `getSpentById()` returnerar 0 för en tom budget

**Andra testet - Budget med utgift**

```php
Spending::factory()->create([
    'space_id' => $this->spaceId,
    'tag_id' => $tag->id,
    'happened_on' => date('Y-m-d'),
    'amount' => 123
]);
```

- Skapar en ny utgift kopplad till budgeten
- Utgiften är daterad till dagens datum
- Belopp sätts till 123 (representerar t.ex. 1.23 i verkliga pengar)

**Verifierar resultatet**

```php
$this->assertEquals(123, $this->budgetRepository->getSpentById($budget->id));
```

- Kontrollerar att repository:t korrekt beräknar totala utgifter
- Testet misslyckas om:
  - Utgiften inte sparats korrekt
  - Beräkningen av totala utgifter är felaktig
  - Repositoryt inte hämtar rätt data från databasen

Detta test är extra viktigt eftersom det verifierar en central del av
applikationens ekonomifunktionalitet - att korrekt spåra och summera utgifter
inom en budget.

# Testning av Policies

> [!NOTE] Policy Testing Policy-tester säkerställer att applikationens
> auktoriseringsregler fungerar korrekt, det vill säga att användare endast kan
> utföra de åtgärder de har behörighet till.

Ett exempel på policy-testning finns i `tests/Feature/SpendingTest.php`:

```php
public function testUnauthorizedUserCantDeleteSpending()
{
    $user = User::factory()->create();
    $space = Space::factory()->create();
    $spending = Spending::factory()->create([
        'space_id' => $space->id
    ]);

    $this->actingAs($user);
    $response = $this->delete('/spendings/' . $spending->id);
    $response->assertStatus(403);
}
```

**Förbereder testmiljön**

```php
$user = User::factory()->create();
$space = Space::factory()->create();
```

- Skapar en ny testanvändare med factory-metoden
- Skapar ett nytt space som kommer innehålla utgiften
- ❗️ Observera att användaren _inte_ är kopplad till space:et, vilket gör dem
  obehöriga

**Skapar testobjektet**

```php
$spending = Spending::factory()->create([
    'space_id' => $space->id
]);
```

- Skapar en utgift kopplad till space:et
- Använder factory för att generera testdata
- Sparar utgiften i testdatabasen

**Simulerar en autentiserad användare**

```php
$this->actingAs($user);
```

- Loggar in den skapade användaren i systemet
- `actingAs()` är en Laravel-metod som simulerar en inloggad användarsession
- Alla kommande requests i testet kommer göras som denna användare

**Försöker radera utgiften**

```php
$response = $this->delete('/spendings/' . $spending->id);
```

- Skickar en DELETE-request till API:et
- Försöker radera en utgift som tillhör ett space användaren inte har tillgång
  till
- Fångar serverns svar i `$response`-variabeln

**Verifierar säkerhetspolicyn**

```php
$response->assertStatus(403);
```

- Kontrollerar att servern svarar med HTTP-status 403 (Forbidden)
- 403 indikerar att användaren är autentiserad men saknar behörighet
- Testet misslyckas om:
  - Servern tillåter borttagning (status 200)
  - Servern svarar med fel felkod
  - Policy:n inte aktiveras korrekt

Detta test är viktigt för applikationens säkerhet eftersom det verifierar att:

1. Policies korrekt blockerar obehöriga åtgärder
2. API:et respekterar policy-beslut
3. Användare inte kan manipulera data som tillhör andra användares spaces

Liknande tester finns för andra policies i projektet, till exempel i
`EarningPolicy`, `TagPolicy` och `SpacePolicy`. Tillsammans skapar dessa en
verifiering av applikationens säkerhetsregler.

# Testning av API

> [!NOTE] API Testing API-tester verifierar att applikationens externa
> gränssnitt fungerar korrekt, inklusive autentisering, datahantering och
> felhantering.

Ett exempel på API-testning finns i
`tests/Feature/Api/TransactionControllerTest.php`. Här är ett av de mer
omfattande testerna:

```php
public function testStoreEndpoint(): void
{
    $user = User::factory()->create();
    $space = Space::factory()->create();
    $user->spaces()->attach($space->id);
    $apiKey = ApiKey::factory()->create(['user_id' => $user->id]);

    $response = $this->postJson(
        '/api/transactions',
        [
            'type' => 'earning',
            'happened_on' => '2021-01-01',
            'description' => 'Helping grandma',
            'amount' => 10.50,
        ],
        [
            'api-key' => $apiKey->token,
        ],
    );

    $response->assertStatus(200);

    $this->assertDatabaseHas(
        Earning::class,
        [
            'happened_on' => '2021-01-01',
            'description' => 'Helping grandma',
            'amount' => 1050,
        ],
    );

    $this->assertDatabaseHas(
        Activity::class,
        [
            'space_id' => $space->id,
            'user_id' => $user->id,
            'entity_type' => 'earning',
            'action' => 'transaction.created',
        ],
    );
}
```

**Förbereder testmiljön**

```php
$user = User::factory()->create();
$space = Space::factory()->create();
$user->spaces()->attach($space->id);
```

- Skapar en testanvändare och ett space
- Kopplar användaren till space:et genom `attach()`
- Detta simulerar en verklig användare med tillgång till ett space

**Skapar API-autentisering**

```php
$apiKey = ApiKey::factory()->create(['user_id' => $user->id]);
```

- Genererar en API-nyckel för användaren
- API-nyckeln kommer användas för att autentisera API-anropet
- Detta simulerar hur externa system skulle interagera med API:et

**Skickar API-request**

```php
$response = $this->postJson(
    '/api/transactions',
    [
        'type' => 'earning',
        'happened_on' => '2021-01-01',
        'description' => 'Helping grandma',
        'amount' => 10.50,
    ],
    [
        'api-key' => $apiKey->token,
    ],
);
```

- Skickar en POST-request till transaktions-endpointen
- Inkluderar testdata för en ny inkomst
- Skickar med API-nyckeln i request headers
- Använder Laravel's `postJson()` för att simulera en JSON API-request

**Verifierar API-svaret**

```php
$response->assertStatus(200);
```

- Kontrollerar att API:et svarar med HTTP-status 200 (OK)
- Verifierar att transaktionen accepterades
- Testet misslyckas om API:et returnerar någon annan statuskod

**Kontrollerar databasen**

```php
$this->assertDatabaseHas(
    Earning::class,
    [
        'happened_on' => '2021-01-01',
        'description' => 'Helping grandma',
        'amount' => 1050,
    ],
);
```

- Verifierar att transaktionen sparades korrekt i databasen
- Kontrollerar alla väsentliga fält
- Notera att beloppet sparas i öre (1050 = 10.50)

**Verifierar aktivitetsloggning**

```php
$this->assertDatabaseHas(
    Activity::class,
    [
        'space_id' => $space->id,
        'user_id' => $user->id,
        'entity_type' => 'earning',
        'action' => 'transaction.created',
    ],
);
```

- Kontrollerar att en aktivitetspost skapades
- Verifierar att aktiviteten är korrekt kopplad till användare och space
- Säkerställer att rätt typ av aktivitet loggades

Sammanfattningsvis så testar detta testet att:

1. Autentisering fungerar korrekt med API-nycklar
2. Data kan tas emot och behandlas korrekt
3. Transaktioner sparas med rätt format och värden
4. Sidoeffekter som aktivitetsloggning fungerar som förväntat

# Mock Testning

> [!NOTE] Mock Testing Mock-testning används när vi vill isolera den kod vi
> testar från dess beroenden. Istället för att använda riktiga objekt skapar vi
> "mockade" versioner som simulerar det förväntade beteendet.

Ett exempel på mock-testning finns i
`tests/Feature/SendVerificationMailTest.php`. Detta test kontrollerar
e-postverifieringsfunktionaliteten:

```php
public function testSuccessfullySent(): void
{
    $firstVerificationMailSentAt = date('Y-m-d H:i:s', strtotime('7 minute ago'));

    $user = User::factory()->create([
        'verification_token' => 'abc123',
        'last_verification_mail_sent_at' => $firstVerificationMailSentAt
    ]);

    // Förhindrar mailet från att sändas
    Mail::fake();

    (new SendVerificationMailAction())->execute($user->id);

    $this->assertGreaterThan(
        $firstVerificationMailSentAt,
        User::find($user->id)->last_verification_mail_sent_at
    );
}
```

**Förbereder testdata**

```php
$firstVerificationMailSentAt = date('Y-m-d H:i:s', strtotime('7 minute ago'));

$user = User::factory()->create([
    'verification_token' => 'abc123',
    'last_verification_mail_sent_at' => $firstVerificationMailSentAt
]);
```

- Skapar en tidsstämpel för "7 minuter sedan"
- Skapar en testanvändare med en verifieringstoken
- Sätter användarens senaste e-postutskick till 7 minuter sedan

**Skapar mock för e-postsystemet**

```php
Mail::fake();
```

- Ersätter Laravel's riktiga e-posthanterare med en mock-version
- Förhindrar att verkliga e-postmeddelanden skickas under testet
- Låter oss verifiera att e-postfunktioner kallas korrekt

**Utför den testade åtgärden**

```php
(new SendVerificationMailAction())->execute($user->id);
```

- Anropar den faktiska funktionen som testas
- Skickar användar-ID som parameter
- Funktionen ska skicka ett verifieringsmail och uppdatera tidsstämpeln

**Verifierar resultatet**

```php
$this->assertGreaterThan(
    $firstVerificationMailSentAt,
    User::find($user->id)->last_verification_mail_sent_at
);
```

- Kontrollerar att tidsstämpeln för senaste mailutskick har uppdaterats
- Verifierar att den nya tidsstämpeln är senare än den ursprungliga
- Testet misslyckas om:
  - Tidsstämpeln inte uppdateras
  - Tidsstämpeln är äldre än originalet
  - Användaren inte kan hittas

Detta test visar:

1. **Isolering**: Genom att mocka e-postsystemet kan man testa koden utan att
   vara beroende av ett fungerande e-postsystem.

2. **Kontroll**: Man kan verifiera att rätt metoder kallas utan att oroa sig för
   sidoeffekter (som att skicka riktiga e-postmeddelanden).

3. **Prestanda**: Mockning gör testerna snabbare eftersom man inte behöver vänta
   på långsamma operationer som e-postutskick.

# Exploratory Testing

> [!NOTE] Exploratory Testing Exploratory testing är en form av testning där
> testaren utforskar applikationen aktivt och kreativt, utan att följa ett
> strikt testmanus. Målet är att hitta buggar och problem som kanske inte skulle
> upptäckas genom mer strukturerad testning.

Jag genomförde detta så gott jag kunde, jag fick snabbt lära mig att det var
tidskrävande när man samtidigt ville dokumentera allt. Jag hittade en hel del
som fungerade men också saker som jag skulle hanterat annorlunda som utvecklare:

Det första jag upptäckte var efter jag loggat in och gick in på fliken
`/budgets` och detta gav en 500 Internal Server Error på /budgets

---

**Orsak:**

1. **Data-Kod Missmatch:**
   - I databasen fanns budgets med `period = 'hourly'`.
   - I koden (BudgetRepository) fanns inte 'hourly' definierad som en giltig
     period.
   - Detta skapade en konflikt när systemet försökte hantera dessa budgets.
2. **Felkedja:**
   - Felet uppstod när systemet försökte bearbeta felaktiga data.
3. **Ursprung till Problemet:**
   - Ett test i `BudgetRepositoryTest.php` skapade budgets med
     `period = 'hourly'`.
   - Dessa budgets rensades inte bort efter testkörning.
   - Datan persisterade i utvecklingsdatabasen och skapade konflikter med koden.

---

**Lösning:**

1. **Kortsiktig Fix:**
   - Rensade databasen från `hourly` budgets.
   - Återställde databasen till ett rent tillstånd.
2. **Långsiktig Prevention (förslagsvis):**
   - Implementerade `RefreshDatabase`-trait i testerna.
   - Detta garanterar att testdata inte persisterar efter körning.
   - Säkerställde att utvecklingsdatabasen hålls ren från test-artefakter.

**Övriga upptäckta problem och observationer**

1. **E-posthantering**

   - Systemet accepterar specialtecken i e-postadresser
   - Stöd för `user+test@example.com` finns
   - Stöd för ovanliga toppdomäner som `.museum` fungerar
   - Problem med hantering av svenska tecken (å,ä,ö)

2. **Filhantering**

   - 413-fel (Request Entity Too Large) vid stora filer
   - Ingen användarvänlig felhantering
   - Saknar tydlig indikation på filstorleksgränser

3. **Säkerhetstestning**

   - XSS-försök i transaktionsnamn hanteras korrekt
   - Lösenordspolicy är svag (accepterar enbokstavslösenord)
   - Ingen kontospärr vid upprepade felaktiga inloggningsförsök

4. **Valideringar och gränssnitt**

   ```plaintext
   Test: Inmatning av ogiltigt datum (2024-12-34)
   Resultat: Formulär nollställs utan felmeddelande
   Förväntat: Tydligt felmeddelande till användaren
   ```

5. **Språkhantering**

   - Inkonsekvent översättning
   - Vissa menyer förblir på engelska vid språkbyte
   - Exempel: "Filter by tag" översätts inte till danska

6. **CSV-import**
   ```plaintext
   Test: Import av transaktioner med negativa belopp (=utgifter)
   Resultat: Felmeddelande "The amount field format is invalid"
   Problem: 500-fel vid import efter borttagning av minustecken
   ```

Detta exploratory testing avslöjade flera viktiga områden som behöver
förbättras, i min mening:

1. **Felhantering**

   - Behov av bättre felmeddelanden
   - Konsekvent felhantering över hela applikationen
   - Användarvänliga felmeddelanden på rätt språk

2. **Validering**

   - Striktare lösenordspolicy behövs
   - Tydligare feedback vid felaktig inmatning
   - Bättre hantering av specialfall

3. **Internationalisering**

   - Komplett översättning av alla strängar
   - Bättre hantering av specialtecken
   - Konsekvent språkhantering

4. **Filhantering**
   - Tydliga gränser för filstorlekar
   - Bättre felmeddelanden vid överskriden filstorlek
   - Stöd för olika filformat

# Test Coverage

![[Pasted image 20250102152737.png]]

- **Klasser:** 22,81% (26 av 114 klasser)
- **Metoder:** 35,35% (105 av 297 metoder)
- **Kodrader:** 39,88% (711 av 1783 rader)

Det är tydligt att projektet har låg testtäckning överlag. Ser man på _Legend_
så räcker det inte ens till medium när det kommer till testtäckning. Eftersom
detta är en finansiell applikation så kan man ju tänka att det dessutom borde
ligga på high, även om den inte hanterar pengar på det sättet.

### Fördjupning i testtäckningen

**Actions-lagret** visar en ganska splittrad bild: Vissa actions, som
_AcceptSpaceInviteAction_ och _CreateSpaceAction_, har 100% täckning. Andra, som
_CreateDefaultWidgetsAction_ och _StoreSpaceInSessionAction_, har 0%.

**Repository-lagret** har bättre täckning, vilket är positivt eftersom det
hanterar affärslogik:

- _BudgetRepository:_ 83,05%
- _RecurringRepository:_ 93,44%
- _SpendingRepository:_ 80%

**Controller-lagret** är har betydligt sämre täckning:

- Många controllers har 0% täckning.
- Endast ett fåtal, som _TranslationsController_, når 100%.
- API-controllers har generellt mycket låg täckning. Kan detta bero på valet av
  designmönster? Att man har valt att arbeta med repository-patern? När man
  kollar in Controllers så är de faktiskt fyllda av beroenden vilket såklart gör
  det betydligt svårare att testa. Detta är ju något som skulle gå att undvika
  genom TDD.

**Mail-klasserna** är däremot imponerande:

- Samtliga har 100% täckning, exempelvis _InvitedToSpace_, _PasswordChanged_ och
  _ResetPassword_.

**Model-lagret** är mer varierande:

- _Budget:_ 80%
- _SpaceInvite:_ 83,33%
- Flera andra modeller ligger under 30%.

**Event-hanteringen** är minst sagt låg den också:

- De flesta event-klasser har 0% eller väldigt låg täckning.
- Endast _TransactionCreated_ når 100%.

### Kategorisering av täckningen

- **Utmärkt täckning (90–100%)**: Mail-klasser, _RecurringRepository_, vissa
  actions.
- **God täckning (70–89%)**: _BudgetRepository_, _SpendingRepository_, modeller
  som _SpaceInvite_.
- **Bristfällig täckning (Under 40%)**: Majoriteten av controllers,
  event-hantering, många modeller och API-endpoints.

### Testresultat

**Totalt antal tester**: 92 **Lyckade tester**: 83 **Fel**: 3 **Misslyckade
assertions**: 6 **Assertions**: 157

Efter analys kunde jag se att det finns två grundläggande problem som orsakar en
dominoeffekt av testfel:

1. Problem med unika värden i databasen: När testerna körs försöker flera tester
   skapa poster med samma unika värden, vilket strider mot databasens
   begränsningar. Detta syns tydligt i felet
   `Duplicate entry 'johndoe@gmail.com' for key 'users.users_email_unique'`.
   Detta påverkar både `LogInController`-testerna och
   `BudgetRepository`-testerna. När ett test försöker skapa en användare med en
   e-postadress som redan finns, kraschar testet.

2. Problem med CSRF. När jag körde testerna stötte jag på ett återkommande
   problem där många tester misslyckades med felkod 419. I Laravel betyder denna
   felkod "Page Expired" och visar att CSRF-skyddet (Cross-Site Request Forgery)
   blockerade våra testanrop. Detta är ett vanligt förekommande fel.

Efter felsökning och åtgärder har testresultaten förbättrats markant:

**Slutgiltigt testresultat:**

- Totalt antal tester: 93
- Lyckade tester: 91 (97.8%)
- Misslyckade assertions (Failures): 2
- Totala assertions: 163

Förbättringen uppnåddes genom följande åtgärder:

1. Identifiering och åtgärd av CSRF-relaterade fel genom implementering av
   `WithoutMiddleware` trait i berörda testfiler
2. Rensning (manuell) av test-databasen efter varje test-körning
3. Säkerställande av korrekt databashantering mellan testkörningar

De två fel som kvarstår beror på kompabilitetskonflikt mellan PHP 8.2 och
PHPUnit 9.6

# Sammanfattning

Det var värdefullt att göra denna typen av uppgift, och det var också en rolig
upplevelse att få "leka detektiv" under exploratory testing. Samtidigt har det
varit tidskrävande, särskilt när man inte är helt bekant med koden och måste
lägga tid på att analysera systemets beteende.

En reflektion är att projektet skulle ha vunnit på att ha en högre andel tester
och bättre täckning (coverage). Om fler tester hade inkluderat "icke happy
paths" – alltså scenarier där saker går fel – hade designen troligen förbättrats
och många av de svagheter jag upptäckt sannolikt undvikits.

Jag har fortfarande vissa oklarheter, särskilt när det gäller de buggar jag
stötte på under granskningen. Ett tydligt exempel är 500-felet som uppstod när
jag försökte nå routern `/budgets`. Efter felsökning visade det sig att
problemet berodde på att testdata från `BudgetRepositoryTest` hade persisterat i
utvecklingsdatabasen. Specifikt hade ett test, som verifierar hanteringen av
ogiltiga perioder, skapat en budget med perioden `hourly`. Denna budget låg kvar
i databasen eftersom systemet inte nollställer databasen mellan testkörningar.

Samma grundproblem verkar också orsaka att vissa tester misslyckas på grund av
duplicerade data. Att inte nollställa databasen mellan tester verkar vara ett
medvetet val, men jag har svårt att förstå logiken bakom det. För mig framstår
det som en enkel och självklar lösning att återställa databasen inför varje
testkörning, men jag misstänker att det finns en bakomliggande anledning som jag
ännu inte har greppat.

Jag har skickat mina frågor och observationer till projektets grundare men har i
skrivande stund inte fått något svar. Jag hoppas dock att en dialog kan leda
till en förståelse för de val som gjorts i projektet.

Det finns även mycket positivt att lyfta fram med projektet. Mappstrukturen var
tydlig och välorganiserad, vilket gjorde det enkelt att navigera och snabbt
förstå vilka tester som testade vad. Detta är något jag uppskattade och som jag
tycker bidrar till projektets läsbarhet. Dessutom fick jag möjlighet att göra en
djupdykning i Docker, vilket kanske inte var kursens huvudsakliga syfte, men det
är definitivt en erfarenhet jag kommer ha nytta av i framtiden. Att få lära mig
mer om Laravel och PHP har också varit väldigt nyttigt, särskilt att läsa och
förstå andras kod. Det är alltid en utmaning, men det har också varit en lärorik
erfarenhet.

Även om jag inte helt förstår valet av repository-designmönstret, så uppskattar
jag att ett mönster faktiskt följdes. Det ger struktur och underlättar
förståelsen för projektet som helhet. Om jag hade fått bestämma så skulle nästa
naturliga steg vara att uppgradera till Laravel 11 och samtidigt se över valet
av repository-designmönstret. Det känns som att detta mönster har skapat viss
komplexitet i testningen, särskilt med tanke på hur beroendena hanteras i
controllers. Dessutom skulle jag prioritera att dokumentera hur projektet
förväntas hantera testning, både för att hjälpa nya utvecklare att bidra och för
att skapa en mer konsekvent teststrategi.

Slutligen tycker jag att det är viktigt att åtgärda de buggar och inkonsekvenser
som framkommit under exploratory testing och öka testtäckningen innan man börjar
implementera nya funktioner. Med tanke på att detta är en applikation för
finansiell hantering bör testningen ligga på en hög nivå, och jag tror att ett
sådant fokus skulle bidra till att göra systemet både mer stabilt och mer
attraktivt för framtida användare och utvecklare. Jag hade gärna bidragit men
känner väl inte riktigt självförtroendet för det i nuläget. Förhoppningen är
dock att grundaren återkopplar till mig och att man efter en dialog kan bidra
med det jag kan.

EDIT: Efter jag skrev rapporten så skrev jag tre tester för att öka coverage
något på BudgetCntroller:

`user_can_view_budget_index`

- Testar att en inloggad användare kan se budget-översiktssidan
- Verifierar att routen `/budgets` returnerar HTTP 200 (OK)
- Säkerställer att användaren har tillgång till sin `space`

`user_can_view_create_budget_page`

- Testar att en inloggad användare kan se sidan för att skapa ny budget
- Verifierar att routen `/budgets/create` returnerar HTTP 200 (OK)
- Kontrollerar åtkomsträttigheter för skapande av budget

`user_can_store_new_budget`

- Testar att en användare kan skapa en ny budget
- Verifierar att data sparas korrekt i databasen
- Kontrollerar att användaren omdirigeras till index-sidan efter skapande
- Validerar att budget kopplas till rätt tag och `space`

Jag önskar att jag haft mer tid att skriva fler tester såklart.
