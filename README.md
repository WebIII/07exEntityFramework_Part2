# 07exEntityFrameworkDeel2Starter
Oefeningen: Hoofdstuk 7 - Entity Framework Core Part II

In deze oefening wordt verder gebouwd op de SportsStore uit oefening EF - deel1 . In het tweede luik van deze oefening bekijken we:
- Mapping van N:M associaties;
- Overerving;
- Seeding;
- Linq to Entities;
- CRUD operaties. 

## Opgave  
1. Clone github https://github.com/WebIII/07exEntityFramework_Part2 (deze repository);
2. Initialiseer de database;
    - Run de applicatie;
3. Seed de database 
    - Maak de klasse `SportsStoreDataInitializer` aan;
    - Maak volgende objecten (`producten`, `steden`, `klanten`, `order` en `orderlijnen`) aan in de `InitializeData` methode van deze klasse, en dit **enkel als er nog geen** `Product` records in de database aanwezig zijn. Onderstaande bevat de code voor de creatie van de objecten. Zorg ervoor dat deze objecten aan de database worden toegevoegd. 

### Products
```csharp
Product football = new Product("Football", 25, "WK colors"); 
Product cornerflags = new Product("Corner flags", 34.95M, "Give your playing field that professional touch"); 
Product shoes = new Product("Running shoes", 95, "Protective and fashionable"); 
Product surfboard = new Product("Surf board", 275, "A boat for one person"); 
Product kayak = new Product("Kayak", 170, "High quality"); 
Product lifeJacket = new Product("Lifejacket", 49.99M, "Protective and fashionable"); 
Product[] products = { football, cornerflags, shoes, surfboard, kayak, lifeJacket }; 
```

### Cities
```csharp
City gent = new City("9000", "Gent"); 
City antwerpen = new City ("3000", "Antwerpen");
City[] cities = { gent, antwerpen }; 
 ```

### Customers with orders and orderlines 
```csharp
Random r = new Random(); 

for (int i = 1; i < 10; i++) 
{ 
    Customer klant = new Customer("student" + i, "Student" + i, "Jan", "Nieuwstraat 10", cities[r.Next(2)]); 
    if (i <= 5) 
    { 
        Cart cart = new Cart(); 
        cart.AddLine(football, 1); 
        cart.AddLine(cornerflags, 2); 
        klant.PlaceOrder(cart, DateTime.Today, false, klant.Street, klant.City); 
    } 
} 

4.  Voeg code toe in `Program.cs` zodat de `InitializeData` methode wordt uitgevoerd. 
Print ter controle alle producten af;
```csharp
var products = context.Products.ToList(); 
products.ForEach(c => Console.WriteLine(c.Name));  
```
5. Run de applicatie;  
6. Commit.

---

### Klasse Category 
Deze klasse dien je verder te implementeren:
1. De associatie met klasse `Product` moet worden toegevoegd. Een `category` kan meerdere `producten` bevatten en een `product` kan tot meerdere `categoriën` behoren. Zorg ervoor dat de associatie achteraf als `many-to-many` relatie kan gemapt worden. Voeg hiervoor de nodige properties, klassen toe. De `Category` kent de `producten`, een `product` kent zijn `categoriën` **niet**. 
2. Implementeer de methodes die nog een `NotImplementedException` throwen. Je kan dit testen via de klasse `CategoryTest` in het `Test Project` (**Deze code dien je eerst uit commentaar te plaatsen**). Maak bij implementatie van de methode `FindProduct` gebruik van de `null conditional operator`. Meer op https://msdn.microsoft.com/en-us/library/dn986595.aspx 
3. Implementeer de mapping;
4. Voeg ook een DbSet toe;
5. Run en controleer de database. 
![Product_Category.png](https://github.com/WebIII/portal/raw/master/docs/H07/fig6.png "Product_Category")

6. Voeg in de Initializer de objecten `Soccer` en `Watersports` toe. He object `Soccer` bevat de eerste 3 `producten`,  de 4 laatste behoren tot het object `Watersports`;
7. Voeg deze toe aan de database.

```csharp
Category watersports = new Category("WaterSports"); 
Category soccer = new Category("Soccer"); 
soccer.AddProduct(football); 
soccer.AddProduct(cornerflags); 
soccer.AddProduct(shoes); 
watersports.AddProduct(shoes); 
watersports.AddProduct(surfboard); 
watersports.AddProduct(kayak); 
watersports.AddProduct(lifeJacket); 
```

---

### Overerving 
1. Maak een nieuwe klasse `OnlineProduct` aan en erf van `Product`;
2. Deze klasse bevat 1 property `ThumbNail`, een string van maximaal 100 characters en is verplicht;
3. Pas in de klasse `Category` de methode `AddProduct` aan. Maak nu gebruik van de  4de parameter `thumbnail`. Als de `thumbnail` null is wordt een `Product` aangemaakt anders een `OnlineProduct`. 
4. Pas in de `initializer` de `Corner Flags` aan. Dit is een `OnlineProduct`. Voeg een waarde voor de `thumbnail` toe. 
5. Map de overerving, run en controleer de database 

| PK  | Column Name   | Data Type     | Allows NULL |
| --- | ------------- |---------------| ----------- |
| X   | ProductId     | int           | No          |
|     | Description   | nvarchar(MAX) | Yes         |
|     | Name          | nvarchar(100) | No          |
|     | Price         | decimal(18,2) | No          |
|     | Thumbnail     | nvarchar(100) | Yes         |
|     | Discriminator | nvarchar(MAX) | Yes         |

> Merk op ThumbNail is toch Allow Nulls, hoewel je de property verplicht gemaakt hebt. Waarom? 

6. Haal eens de rijen in de tabel Product op. De `Discriminator` kolom bevat ofwel de waarde `Product`, ofwel de waarde `OnlineProduct`.

![Product_Rows.png](https://github.com/WebIII/portal/raw/master/docs/H07/fig7.png "Product_Rows")
 
---

### Linq to Entities en CRUD 
1. Plaats de lijn `DoExercise()` in `Program.cs` uit commentaar;
2. Los de queries op. Maak je queries zo performant mogelijk;
> Merk op: vraag 9 zal aan de client side worden geëvalueerd door gebruik van de methode HasOrdered  

#### Voorbeeld output: (merk op: de datums van de orders kunnen afwijken) 
```
--1. Alle producten gesorteerd op prijs oplopend, dan op naam-- 
1 Football 25,00 
2 Corner flags 34,95 
6 Lifejacket 49,99 
3 Running shoes 95,00 
5 Kayak 170,00 
4 Surf board 275,00 

--2. Alle klanten die wonen te gent, gesorteerd op naam -- 
Student2 Jan 
Student4 Jan 
Student6 Jan 
Student8 Jan 
 
--3. Het aantal klanten in Gent-- 
4 
 
--4. De klant met  customername student4. -- 
Student4 Jan 

--5. Vervolg op vorige query. Print nu de orders van die klant af. Print (methode WriteOrder) de Orderdate, DeliveryDate, en Total af. 
09-Nov-17 12:00:00 AM 09-Nov-17 12:00:00 AM 94.90 

--6. Alle producten van de categorie soccer, gesorteerd op prijs descending-- 
3 Running shoes 95,00 
2 Corner flags 34,95 
1 Football 25,00 

--7A. Maak een nieuwe cart aan en voeg product met id 1 toe(aantal 2). -- 

--7B. Plaatst dan een order voor student 4 voor deze cart, deliverydate binnen 20 dagen, giftwrapping false en deliveryAddress = adres van de klant--. Persisteer in database. Print vervolgens alle orders van de klant. 
1/11/2016 0:00:00 1/11/2016 0:00:00 94,90 
21/11/2016 0:00:00 1/11/2016 0:00:00 50 

--8. Alle klanten met een order met DeliveryDate binnen de 10 dagen, sorteer op naam-- 
Student1 Jan 
Student2 Jan 
Student3 Jan 
Student4 Jan 
Student5 Jan 

--9. Alle klanten die een product met id 1  hebben besteld-- 
Student2 Jan 
Student4 Jan 
Student1 Jan 
Student3 Jan 
Student5 Jan 

--10. Alle klanten met orders, met vermelding van aantal orders. Maak gebruik van een anoniem type-- 
Student2 1 
Student4 2 
Student1 1 
Student3 1 
Student5 1 

--11. Pas de naam aan van klant student5, in je eigen naam en voornaam-- 
Student2 Jan 
Student4 Jan 
Student6 Jan 
Student8 Jan 
Student1 Jan 
Student3 Jan 
MijnNaam MijnVoornaam 
Student7 Jan 
Student9 Jan 

--12A. Verwijder de eerste klant (in alfabetische volgorde van customername) zonder orders-- 
Student2 Jan 
Student4 Jan 
Student8 Jan 
Student1 Jan 
Student3 Jan 
MijnNaam MijnVoornaam 
Student7 Jan 
Student9 Jan 

--13. Maak een nieuw product training, prijs 80, aan die behoort tot de categorie soccer en watersports en print na toevoeging aan de database het productid af-- 
7 

--14. Product training behoort niet langer tot de category soccer. Ga na of CategoryProduct ook verwijderd wordt- 


--15. Probeer de stad Gent te verwijderen. Waarom lukt dit niet?-- 
> Verwijderen Gent faalt. Reason : An error occurred while updating the entries. See the inner exception for details. 
