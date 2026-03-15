# NNPIA LAB 05: Testování Spring aplikace

Cílem tohoto cvičení je seznámit se se základními přístupy k testování backendových aplikací.  
Studenti si vyzkouší implementaci testů s **mockovanými závislostmi** a následně i s **testovací databází** pomocí H2.

---

## Předpoklady

- [JDK 21 nebo novější](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)
    - Na školních počítačích je nainstalována JDK 17. Můžete ale použít funkci IntelliJ IDEA pro stažení a nastavení JDK
      21 přímo v IDE (File → Project Structure → SDKs → + → JDK).

- [IntelliJ IDEA](https://www.jetbrains.com/idea/download/)
    - Doporučena je verze **IntelliJ IDEA Ultimate**
        - Na školíních počítačích je nainstalována verze 2023.3.3.
        - Na osobních počítačích doporučujeme využít nejnovější verzi 2025.2.3 nebo novější.
    - Studenti mohou využít **bezplatnou studentskou licenci**

- [Verzovací systém Git](https://git-scm.com/downloads)

- **Webový prohlížeč** ideálně postavený na jádru Chromium
    - Google Chrome, Microsoft Edge, Brave…

- **HTTP klient pro testování API**
    - [Postman](https://www.postman.com/downloads/)
    - [Insomnia](https://insomnia.rest/download)

- [Docker](https://www.docker.com/)

---

# 5.1. Endpoint pro získání uživatele podle ID

K testování využijeme endpoint **GET `/api/v1/app-users/{id}`** který vrací jednoho uživatele podle jeho id který byl
součástí dobrovolných úkolů v **LAB03**.

![Cvičení](https://img.shields.io/badge/Spring%20Boot-Cvičení-black?style=flat&labelColor=6DB33F&logo=springboot&logoColor=white)

![Cvičení](https://img.shields.io/badge/Docker-Cvičení-black?style=flat&labelColor=2496ED&logo=docker&logoColor=white)

1. Zkontrolujte, zda máte implementovaný endpoint pro získání uživatele podle ID.
2. Pokud endpoint neexistuje, doimplementujte jej v `AppUserController`.
3. Endpoint by měl:
    - přijímat parametr `id` z URL
    - vracet jednoho uživatele
    - vracet odpovídající HTTP status kódy
4. Spusťte aplikaci a pomocí REST klienta ověřte, že endpoint funguje.

💡 _Ujistěte se, že máte spuštěný Docker kontejner._

5. Otestujte alespoň dva scénáře:
    - získání existujícího uživatele
    - požadavek na neexistující ID

💡 _Tyto scénáře budeme později automatizovat pomocí testů._

---

# 5.2. Příprava na testování

Rozdíl mezi typy testů:

- Unit test -> Testuje jednu třídu bez Springu.
- Controller test -> Testuje HTTP endpoint, ale závislosti jsou mockované.
- Integrační test -> Testuje spolupráci více vrstev aplikace včetně databáze.

![Cvičení](https://img.shields.io/badge/H2-Cvičení-black?style=flat&labelColor=1E90FF&logo=databricks&logoColor=white)

1. Vytvořte nový test `AppUserControllerTest` pro ověření funkčnosti endpointu v `src/test/java/controller`.
2. Test by měl mít následující podobu:

```java
package cz.upce.fei.backend.controller;

import org.junit.jupiter.api.Test;

class AppUserControllerTest {

    @Test
    void getAppUserById() {
    }
}
```

💡 _Test můžete jednoduše vytvořit v IntelliJ IDEA kliknutím pravým tlačítkem na třídu `AppUserController` a výběrem "Go
to" → "Test"._

3. Zkuste test spustit:

```terminaloutput
> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :compileTestJava
> Task :processTestResources
> Task :testClasses
> Task :test
BUILD SUCCESSFUL in 1s
5 actionable tasks: 3 executed, 2 up-to-date
Consider enabling configuration cache to speed up this build: https://docs.gradle.org/9.3.0/userguide/configuration_cache_enabling.html
16:00:42: Execution finished ':test --tests "cz.upce.fei.backend.controller.AppUserControllerTest"'.
```

💡 _Všimněte si že během testu se nespustil Spring kontext. V současné podobě se jedná o jednoduchý Unit test. My ale
potřebujeme napsat integrační test pro testování Springu._

4. Opatřete tedy test anotací `@SpringBootTest` a znovu jej spusťte:

```terminaloutput
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v4.0.2)

2026-03-14T16:05:59.686+01:00  INFO 14713 --- [backend] [    Test worker] c.u.f.b.c.AppUserControllerTest          : Starting AppUserControllerTest using Java 21.0.10 with PID 14713 (started by ondychrbi in /Users/ondychrbi/Teaching/nnpia-2026-cviceni-ondyChrbi/backend)
2026-03-14T16:05:59.687+01:00 DEBUG 14713 --- [backend] [    Test worker] c.u.f.b.c.AppUserControllerTest          : Running with Spring Boot v4.0.2, Spring v7.0.3
2026-03-14T16:05:59.687+01:00  INFO 14713 --- [backend] [    Test worker] c.u.f.b.c.AppUserControllerTest          : No active profile set, falling back to 1 default profile: "default"
2026-03-14T16:05:59.688+01:00 DEBUG 14713 --- [backend] [    Test worker] o.s.boot.SpringApplication               : Loading source class cz.upce.fei.backend.BackendApplication
```

💡 _Všimněte si že nyní se v rámci testu spustí i Spring. `@SpringBootTest` zajistí, že se spustí celý kontext aplikace,
což znamená, že se načtou všechny komponenty, služby, konfigurace a další bean objekty definované v aplikaci. Díky tomu
je možné testovat aplikaci v prostředí, které se co nejvíce podobá reálnému běhu aplikace._

5. Zkuste nyní vypnout kontejner s databází v Docker Desktop nebo pomocí příkazu a znovu spusťte test:

```bash
docker stop nnpia-postgres
```

```terminaloutput
org.postgresql.util.PSQLException: Connection to localhost:5432 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.
	at org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:373) ~[postgresql-42.7.9.jar:42.7.9]
```

💡 _Podle logu se test pokouší připojit k PostgreSQL databázi, kterou jsme nakonfigurovali v předchozím LABu._

6. Přidejte do projektu závislost pro databázi H2.

```groovy
testImplementation 'com.h2database:h2'
```

💡 _H2 je in-memory databáze, která je ideální pro testování. Její obsah je uložen v operační paměti a je izolovaný pouze
pro testy. Testovací databáze umožňuje spouštět testy nezávisle na vývojářském prostředí._

7. Vytvořte nový konfigurační soubor `application.properties` v `src/test/resources`.
    - Pokud IntelliJ složku automaticky nerozpozná, označte složku jako Test Resources Root. (klikněte pravým tlačítkem
      na složku → "Mark Directory as" → "Test Resources Root").
    - Testovací třídu opatřete anotací `@ActiveProfiles("test")` aby se načítala konfigurace z `application.properties`
      v testovacím prostředí.

8. V `application.properties` v `src/test/resources` nastavte H2 databázi tak, aby ukládala data do operační paměti.

```properties
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
spring.jpa.hibernate.ddl-auto=update
```

9. Restartujte test a ověřte že proběhl úspěšně a připojil se k H2 databázi.

```terminaloutput
...
2026-03-15T11:55:36.606+01:00  INFO 27763 --- [    Test worker] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2026-03-15T11:55:36.636+01:00  INFO 27763 --- [    Test worker] org.hibernate.orm.connections.pooling    : HHH10001005: Database info:
	Database JDBC URL [jdbc:h2:mem:testdb]
	Database driver: H2 JDBC Driver
	Database dialect: H2Dialect
...
```

---

# 5.3. Unit test controlleru s mockovanou závislostí

Při testování aplikací se často dostáváme do situace, kdy testovaná logika závisí na dalších komponentách nebo na API
dalších služeb či třetích stran. Tyto lze závislosti nahradit tzv. **mock objekty**, které simulují chování skutečných komponent.

V této části vytvoříme test, ve kterém bude **AppUserRepository mockováno**.

![Cvičení](https://img.shields.io/badge/Spring%20Boot-Cvičení-black?style=flat&labelColor=6DB33F&logo=springboot&logoColor=white)

1. Opatřete testovací třídu `AppUserControllerTest` anotací `@AutoConfigureMockMvc` která nám umožní testovat HTTP
   endpointy.
    - Do testovací třídy přidejte atribut `MockMvc` s anotací `@Autowired` pro injektování této závislosti.

```java
@Autowired
private MockMvc mockMvc;
```

2. Nakonfigurujte test tak, aby místo skutečné repository používal **mock objekt**.
    - Test pojmenujte smysluplně aby název odrážel testovanou funkcionalitu. Např.:
      `getAppUserById_existingUser_return200_responseBody`.
    - Využijte anotaci `@MockitoBean` pro vytvoření mocku `AppUserRepository` a injektujte `MockMvc` pro testování HTTP
      endpointů.

```java
@MockitoBean
private AppUserRepository appUserRepository;
```

3. Nastavte mock chování pro `AppUserRepository` pomocí `Mockito` tak aby:
    - Při hledání uživatele pomocí metody `findById` s parametrem `1L` vrátil existujícího uživatele.
    - Při hledání uživatele pomocí metody `findById` s parametrem `2L` vrátil `Optional.empty()`, což bude simulovat
      neexistujícího uživatele.

```java
@Test
public void getAppUserById_existingUser_return200_responseBody() throws Exception {
    var existingAppUser = AppUser.builder()
            .id(1L)
            .email("random@upce.cz")
            .password("testPassword")
            .active(true)
            .hashtags(new HashSet<>())
            .build();

    when(appUserRepository.findById(existingAppUser.getId()))
            .thenReturn(Optional.of(existingAppUser));
}
```

💡 _Mockování si můžete představit jako podvrženou instanci třídy která simuluje chování určené programátorem._

4. Implementujte logiku pro úspěšné získání existujícího uživatele (ID `1L`).
    - Test ověří že návratový status je `200 OK`.
    - Response body obsahuje správné údaje o uživateli (`id`, `email`, `active`).

```java

@Test
public void getAppUserById_existingUser_return200_responseBody() throws Exception {
    var existingAppUser = AppUser.builder()
            .id(1L)
            .email("random@upce.cz")
            .password("testPassword")
            .active(true)
            .hashtags(new HashSet<>())
            .build();

    when(appUserRepository.findById(existingAppUser.getId()))
            .thenReturn(Optional.of(existingAppUser));

    mockMvc.perform(get("/api/v1/app-users/" + existingAppUser.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(existingAppUser.getId()))
            .andExpect(jsonPath("$.email").value(existingAppUser.getEmail()))
            .andExpect(jsonPath("$.active").value(existingAppUser.getActive()));
}
```

5. Test pro získání neexistujícího uživatele (ID `2L`).
    - Test ověří že návratový status je `404 NOT FOUND` pro neexistujícího uživatele.

---

# 5.4. Integrační test s H2 databází

Mockování není vždy ideální přístup pro testování, protože neodráží skutečné chování aplikace.
Jako programátoři musíme některé mockované části sami definovat a může se snadno stát, že náš mock nebude přesně
odpovídat skutečnému chování. Nyní upravíme test vytvořený v předchozí části tak, aby místo mocku používal
**testovací databázi H2**.

![Cvičení](https://img.shields.io/badge/Spring%20Boot-Cvičení-black?style=flat&labelColor=6DB33F&logo=springboot&logoColor=white)

1. Nahraďte anotaci `@MockitoBean` anotací `@Autowired` pro `AppUserRepository` abyste využili skutečnou implementaci
   repository.
2. Upravte testy tak, aby místo mockovaného chování využívaly reálná testovací data.

```java
@Test
@Transactional
public void getAppUserById_existingUser_return200_responseBody() throws Exception {
    var existingAppUser = AppUser.builder()
            .email("random@upce.cz")
            .password("testPassword")
            .active(true)
            .hashtags(new HashSet<>())
            .build();

    existingAppUser = appUserRepository.save(existingAppUser);

    mockMvc.perform(get("/api/v1/app-users/" + existingAppUser.getId()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(existingAppUser.getId()))
            .andExpect(jsonPath("$.email").value(existingAppUser.getEmail()))
            .andExpect(jsonPath("$.active").value(existingAppUser.getActive()));
}
```

3. Spusťte test a ověřte, že komunikuje s databází H2.

💡 _Takový test se označuje jako **integrační test**, protože testuje spolupráci více komponent aplikace bez nutnosti
mockování._

---

# 5.5. Rozšíření integračních testů

Protože implementace dalších testů je velmi podobná, můžete si jejich vytvoření usnadnit pomocí AI asistenta.

![Vibe coding – Dobrovolný úkol](https://img.shields.io/badge/Vibe%20coding-Dobrovolný%20úkol-black?style=flat&labelColor=800020&logo=githubcopilot&logoColor=white)

1. Pomocí funkce `Agent` vytvořte prompt, který rozšíří třídu `AppUserControllerTest` o další integrační testy.
2. Pomocí hashtagu označte v promptu všechy třídy a soubory které souvisí s testovanou logikou.
3. Po vygenerování testů zkontrolujte, že:
    - testy procházejí
    - ověřují správné HTTP status kódy
    - ověřují obsah odpovědi endpointu

---

## Odevzdání

- Po dokončení všech úkolů vytvořte **commit** se všemi provedenými změnami.
- Commit **pushněte do vzdáleného repozitáře**.
- Název commitu musí začínat označením **LAB05**.

**Příklad commit message:**

```git
LAB05 - Testování aplikace

Unit test controlleru s mockovanou závislostí.
Integrační test s H2 databází.
```

---

## Užitečné odkazy a zdroje

- [Unit Test, Mocking and Code Coverage in Spring Boot](https://medium.com/@prasanta-paul/unit-test-mocking-and-code-coverage-in-spring-boot-26f8aa8a2554)
- [Configuring Separate Spring DataSource for Tests](https://www.baeldung.com/spring-testing-separate-data-source)
- [DevOps CI/CD Explained in 100 Seconds](https://www.youtube.com/watch?v=scEDHsr3APg)
- [Programování webových aplikací - vytváření testovacích relačních dat](https://www.youtube.com/watch?v=XGUDFN_OTPs)
- [Programování webových aplikací - Vytváření selenium testů](https://www.youtube.com/watch?v=vnF88s74k4k)
- [Programování webových aplikací - Continuous Integration](https://www.youtube.com/watch?v=QjHqOJCOZvI)