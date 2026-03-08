# NNPIA LAB 04: Spring Data JPA

Cílem tohoto cvičení je seznámit se s perzistencí dat v relační databázi. Pro ukládání dat použijeme databázi **PostgreSQL** v Docker kontejneru. Pro přístup k databázi bude využita technologie Spring Data JPA.

---

## Předpoklady

- [JDK 21 nebo novější](https://www.oracle.com/java/technologies/javase/jdk21-archive-downloads.html)
    - Na školních počítačích je nainstalována JDK 17. Můžete ale použít funkci IntelliJ IDEA pro stažení a nastavení JDK
      21 přímo v IDE (File → Project Structure → SDKs → + → JDK).

- [IntelliJ IDEA](https://www.jetbrains.com/idea/download/)
    - Doporučena je verze **IntelliJ IDEA Ultimate**
        - Na školních počítačích je nainstalována verze 2023.3.3.
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

## 4.1. Příprava relační databáze

V tomto cvičení budeme využívat relační databázi **PostgreSQL**. Díky kontejnerizaci dnes vývojáři nemusí instalovat databázi přímo do operačního systému. Databáze běží izolovaně v kontejneru, což zajišťuje reprodukovatelnost prostředí.

![Cvičení](https://img.shields.io/badge/Docker-Cvičení-black?style=flat&labelColor=2496ED&logo=docker&logoColor=white)

![Cvičení](https://img.shields.io/badge/PostgreSQL-Cvičení-black?style=flat&labelColor=336791&logo=postgresql&logoColor=white)

1. Otevřete terminál přímo v IntelliJ IDEA.
2. Ověřte, že Docker běží pomocí příkazu:

```bash
docker version
```

```terminaloutput
Client:
 Version:           28.5.1
 API version:       1.51
 Go version:        go1.24.8
...
```

💡 _Výstup se bude lišit podle verze kterou máte nainstalovanou._

3. Stáhněte image PostgreSQL:

```bash
docker pull postgres:latest
```

```terminaloutput
latest: Pulling from library/postgres
e0efe371a6b4: Download complete 
9ecd997d0dc2: Download complete 
fb7a6ccc5f55: Download complete 
09b5a489e410: Download complete 
0cd0fdf3766b: Download complete 
001ce9848f5a: Download complete 
996ccc8b94c2: Downloading [>                                                  ]  1.049MB/113.9MB
ff75d590949d: Downloading [========>                                          ]  1.049MB/6.231MB
897f6ad82b48: Downloading [======>                                            ]  1.049MB/8.204MB
b8877a0d854a: Download complete 
...
Status: Downloaded newer image for postgres:latest
```

4. Ověřte, že image byl stažen v Docker Desktop nebo pomocí příkazu:

```bash
docker images
```

```terminaloutput
REPOSITORY                TAG       IMAGE ID       CREATED       SIZE
postgres                  latest    b6b4d0b75c69   9 days ago    665MB
```

5. Spusťte kontejner:

```bash
docker run --name nnpia-postgres \
-e POSTGRES_DB=nnpia \
-e POSTGRES_USER=nnpia \
-e POSTGRES_PASSWORD=nnpia \
-p 5432:5432 \
-v nnpia-data:/var/lib/postgresql \
-d postgres:latest
```

```terminaloutput
bfe85cd9b868376d9f760653e7b24d71d890311aa6e2abf44d6d947bc4663c1a
```

💡 _Výstup se bude lišit na Vašem zařízení._

❗ Pokud je port 5432 obsazen, použijte jiný (např. 5433).

6. Ověřte, že kontejner běží v Docker Desktop nebo pomocí příkazu:

```bash
docker ps
```

```terminaloutput
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS         PORTS                                         NAMES
b01ce0b87a4f   postgres:latest   "docker-entrypoint.s…"   10 seconds ago   Up 9 seconds   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp   nnpia-postgres

```

7. Ověřte, že volume byl vytvořen v Docker Desktop nebo pomocí příkazu:

```bash
docker volume ls
```

```terminaloutput
DRIVER    VOLUME NAME
local     nnpia-data
```

💡 _Volume je úložiště mimo kontejner. Data databáze zůstávají zachována i po zastavení nebo smazání kontejneru._

---

## 4.2. Připojení pomocí databázového klienta v IntelliJ IDEA

V této části si ověříte funkčnost databáze a schopnost navázat spojení pomocí databázového klienta. Připojení lze provést nejen v IntelliJ IDEA, ale také pomocí nástrojů jako `TablePlus`, `pgAdmin` nebo `DataGrip`.

![Cvičení](https://img.shields.io/badge/PostgreSQL-Cvičení-black?style=flat&labelColor=336791&logo=postgresql&logoColor=white)

1. Otevřete panel **Database** v IntelliJ IDEA.
2. Vytvořte nové připojení typu PostgreSQL.
3. Nastavte host `localhost`, port `5432`, databázi `nnpia`, uživatele `nnpia` a heslo `nnpia`.
    - Připojení vhodně pojmenujte (např. `NNPIA Local Docker Database`).
4. Otestujte připojení a případně potvrďte stažení driveru.

---

## 4.3. Spring Data JPA

Spring Data JPA umožňuje pracovat s databází pomocí objektů místo ručního psaní SQL dotazů.

![Cvičení](https://img.shields.io/badge/Spring%20Boot-Cvičení-black?style=flat&labelColor=6DB33F&logo=springboot&logoColor=white)

1. Přidejte do projektu závislosti pro Spring Data JPA a PostgreSQL:

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'org.postgresql:postgresql'
```

2. Spusťte backend a sledujte logy. Měli byste vidět chybu připojení k databázi, protože jsme ještě nenakonfigurovali datasource:

```terminaloutput
***************************
APPLICATION FAILED TO START
***************************
Description:
Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.
Reason: Failed to determine a suitable driver class
```

💡 _Přidáním nových závislostí se aktivovala funkce Springu které se říká `Autoconfigure`. Spring se pokusil naicializovat beany z této závislosti. Jelikož ale nemá údaje pro připojení k databázi inicializace selhala._

3. Nakonfigurujte připojení k databázi v souboru `application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/nnpia
spring.datasource.username=nnpia
spring.datasource.password=nnpia
```

4. Restartujte aplikaci a sledujte logy.

💡 _Databázi lze spravovat také pomocí `docker-compose.yml` a závislosti `spring-boot-docker-compose`, která umožňuje automatické spuštění kontejnerů při startu aplikace._

---

## 4.4. Povýšení AppUser na databázovou entitu

Aby mohl Spring pracovat s databází pomocí ORM, musí být třída označena jako entita.

![Cvičení](https://img.shields.io/badge/Spring%20Boot-Cvičení-black?style=flat&labelColor=6DB33F&logo=springboot&logoColor=white)

1. Otevřete třídu `AppUser`.
2. Označte ji jako entitu pomocí `@Entity` anotace.
    - Dodatečně můžete použít anotaci `@Table` pro nastavení názvu tabulky, pokud nechcete používat výchozí název
      odvozený z názvu třídy.

💡 _Anotace `@Entity` říká ORM nástroji, že třída reprezentuje databázovou tabulku._

3. Označte atribut `id` jako primární klíč pomocí `@Id` anotace.
    - Nastavte automatické generování hodnoty ID databází pomocí `@GeneratedValue` anotace.

💡 _Anotace `@Id` určuje, který atribut je jednoznačný identifikátor záznamu._

4. Ujistěte se, že třída obsahuje bezparametrický konstruktor.
5. Pokud restartujete aplikaci, zjistíte, že se v databázi nevytvořila tabulka `app_user`.
6. Nastavte generování schématu pomocí vlastnosti `ddl-auto` v `application.properties`:

```properties
spring.jpa.hibernate.ddl-auto=update
```

7. Restartujte aplikaci a sledujte logy. Měli byste vidět, že se vytvořila tabulka `app_user` v databázi:
    - Pomocí database pluginu v IntelliJ IDEA můžete ověřit, že tabulka skutečně existuje.

```terminaloutput
2026-02-27T22:19:57.814+01:00 DEBUG 59820 --- [backend] [  restartedMain] org.hibernate.SQL                        : create table app_user (id bigint generated by default as identity, active boolean, email varchar(255), password varchar(255), primary key (id))
```

💡 _Vlastnost `spring.jpa.hibernate.ddl-auto=update` je vhodná především pro vývoj. V produkčních aplikacích se změny databázového schématu obvykle spravují pomocí nástrojů jako Flyway nebo Liquibase, které umožňují verzování databázových migrací._

---

## 4.5. Přesměrování GET a POST endpointu

V následujících krocích postupně přesměrujeme všechny endpointy z dočasné in-memory kolekce na JPA repository, která bude komunikovat s databází. Tím zajistíme trvalé uložení dat a využijeme výhod ORM.

![Cvičení](https://img.shields.io/badge/Spring%20Boot-Cvičení-black?style=flat&labelColor=6DB33F&logo=springboot&logoColor=white)

1. Vložte testovací data pomocí SQL konzole v IntelliJ.

💡 _Použijte volně dostupné AI k vytvoření SQL insert příkazu._

2. Vytvořte rozhraní `AppUserRepository` dědící z `JPARepository` v balíčku `repository`:

```java
public interface AppUserRepository extends JpaRepository<AppUser, Long> {
}
```

💡 _Všimněte si, že rozhraní má dva generické parametry. Jeden představuje databázovou entitu a druhý datový typ jejího primárního klíče._

3. Vložte repository do `AppUserService` vrstvy pomocí konstruktoru.
4. Upravte metodu pro získání všech uživatelů tak, aby používala repository.
    - Použijte metodu `findAll()` z repository, která vrací všechny záznamy z tabulky `app_user`.
    - Opatřete metodu anotací `@Transactional(readOnly = true)`.
    - Otestujte funkčnost pomocí REST klienta.
5. Upravte metodu pro vytvoření uživatele tak, aby ukládala data do databáze.
    - Použijte metodu `save()` z repository, která uloží nový záznam do tabulky `app_user`.
    - Odstraňte atribut pro generování ID, protože databáze nyní automaticky generuje ID.
    - Opatřete metodu anotací `@Transactional`.
    - Otestujte funkčnost pomocí REST klienta.
6. Upravte metodu pro smazání uživatele tak, aby používala repository.
    - Použijte metodu `deleteById()` z repository, která smaže záznam z tabulky `app_user` podle ID.
    - Opatřete metodu anotací `@Transactional`.
    - Upravte kontrolu existence aby využívala metodu `existsById()` z repository.
    - Otestujte funkčnost pomocí REST klienta.

💡 _Rozhraní JpaRepository poskytuje základní databázové operace (CRUD) bez nutnosti psát SQL dotazy. Spring Data JPA automaticky generuje implementaci metod jako findAll(), save(), deleteById() nebo existsById()._

---

## 4.6. DTOs

Nyní třída `AppUser` reprezentuje jak databázovou entitu, tak i datový model pro API. To ale není ideální, protože můžeme nechtěně vystavit citlivá data (např. heslo) nebo být vázáni na strukturu databáze.

![Cvičení](https://img.shields.io/badge/Spring%20Boot-Cvičení-black?style=flat&labelColor=6DB33F&logo=springboot&logoColor=white)

1. Vytvořte třídu `AppUserResponseDto` v balíčku `dto`:
    - Obsahuje pouze pole `id`, `email` a `active`.
2. Vytvořte mapování mezi dto a entitou.
3. Upravte všechny endpointy s návratovým typem `AppUser` tak, aby vraceli DTO místo entity.

💡 _DTO zabraňuje úniku citlivých dat a odděluje databázový model od API kontraktu._

---

## 4.7. Relace

V této části si prakticky vyzkoušíte modelování vztahů mezi entitami pomocí JPA.

![Semestrální práce - Doporučený úkol](https://img.shields.io/badge/Semestrální%20práce-Doporučené%20cvičení-black?style=flat&labelColor=FFD966&color=black&logo=bookstack&logoColor=white)

1. Vytvořte novou entitu `Hashtag`.

Entita bude obsahovat následující atributy:

- `id` – primární klíč
- `name` – název hashtagu
- `createdAt` – datum a čas vytvoření hashtagu

2. V entitě `AppUser` vytvořte vztah na entitu `Hashtag`.

```java
@ManyToMany
@JoinTable(
        name = "app_user_hashtag",
        joinColumns = @JoinColumn(name = "app_user_id"),
        inverseJoinColumns = @JoinColumn(name = "hashtag_id")
)
@EqualsAndHashCode.Exclude
private Set<Hashtag> hashtags;
```

3. V entitě `Hashtag` vytvořte opačnou stranu vztahu.

- implementujte Many-to-Many relaci.
- použijte atribut `mappedBy`.

```java
@ManyToMany(mappedBy = "hashtags")
@EqualsAndHashCode.Exclude
private Set<AppUser> appUsers;
```

💡 _Všimněte si že oba atributy mají dodatečnou Lombok anotaci `@EqualsAndHashCode.Exclude`. Zkuste si jí po dokončení sekce odstranit a analyzujte výjimku, která vznikla._

💡 _Many-to-Many relace je v relační databázi realizována pomocí spojovací tabulky (join table). Hibernate tuto tabulku při startu aplikace automaticky vytvoří na základě definice relace v entitách._

4. Restartujte backend aplikaci a sledujte změny v databázi.

5. Pomocí SQL konzole v IntelliJ IDEA vložte několik hashtagů a přiřaďte je uživatelům pomocí spojovací tabulky.

💡 _Použijte volně dostupné AI k vytvoření SQL insert příkazu._

6. Upravte endpointy pro získání uživatelů tak, aby vracely také jejich hashtagy.
- Nezapomeňte na best practice, které jsme si v rámci cvičení ukázali.

---

## 4.9. Přesměrování zbylých endpointů

![Vibe coding – Dobrovolný úkol](https://img.shields.io/badge/Vibe%20coding-Dobrovolný%20úkol-black?style=flat&labelColor=800020&logo=githubcopilot&logoColor=white)

1. Pomocí funkce `Agent` upravte zbývající endpointy aby využívali JPA repository místo in-memory kolekce.
2. Odstraňte in-memory kolekci z `AppUserService`, protože již není potřeba.

---

## Odevzdání

- Po dokončení všech úkolů vytvořte **commit** se všemi provedenými změnami.
- Commit **pushněte do vzdáleného repozitáře**.
- Název commitu musí začínat označením **LAB04**.

```git
LAB04 - Databáze a JPA

Připojení k PostgreSQL v Dockeru, přesměrování endpointů na JPA repository, implementace DTO a relace mezi entitami.
```

---

## Užitečné odkazy a zdroje

- [Spring Component Scanning](https://www.baeldung.com/spring-component-scanning)
- [How Spring Boot Auto-Configuration Works](https://medium.com/@AlexanderObregon/how-spring-boot-auto-configuration-works-68f631e03948)
- [Follow Up #1 — @Id generation strategy in Spring Boot](https://medium.com/womenintechnology/follow-up-1-id-generation-strategy-in-spring-boot-861d939353fc)
- [Hibernate Schema Management: Get It Right, Fast (spring.jpa.hibernate.ddl-auto)](https://medium.com/@rihab.beji099/hibernate-schema-management-get-it-right-fast-spring-jpa-hibernate-ddl-auto-cfd86830daa2)
- [Introduction to Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)
- [Many-To-Many Relationship in JPA](https://www.baeldung.com/jpa-many-to-many)
- [Fetching Strategies in JPA: Optimizing Query Performance](https://www.codingshuttle.com/spring-boot-handbook/fetching-strategies/)
- [Návrhový vzor Proxy](https://refactoring.guru/design-patterns/proxy)