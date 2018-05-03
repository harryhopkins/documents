### Introduction

In this document, we will create, step by step, a sample **phonebook application**
based on ASP.NET Zero (ASP.NET Core & Angular 5 version).
At the end of these steps, we will have a multi-tenant, localized, authorized,
configurable and testable... application.

### Creating And Running The Project

We're creating and downloading the solution named
"**Acme.PhoneBookDemo**" as described in the [Getting
Started](Getting-Started-Angular.md) document. Please follow the
getting started document, run the application, login as default tenant
admin (select Default as tenancy name, use admin as username and 123qwe
as password) and see the dashboard below:

<img src="images/dashboard1.png" alt="Dashboard" style="width:100.0%" />

Logout from the application for now. We will make our application
**single-tenant** (we will convert it to multi-tenant later). So, we
open **PhoneBookDemoConsts** class in the **Acme.PhoneBookDemo.Core**
project and disable multi-tenancy as shown below:

    public class PhoneBookDemoConsts
    {
        public const string LocalizationSourceName = "PhoneBookDemo";

        public const string ConnectionStringName = "Default";

        public const bool MultiTenancyEnabled = false;

        public const int PaymentCacheDurationInMinutes = 30;
    }

**Note:** If you try logging in before changing **MultiTenancyEnabled** to
false, you might be get a login error. To resolve this, you should remove
cookies associated with the new application.

### Adding a New Menu Item

Let's begin with the UI and create a new page named "**Phone book**".

#### Defining a Menu Item

Open **src\\app\\shared\\layout\\nav\\app-navigation.service.ts** in the
client side project (**Acme.PhoneBookDemo.AngularUI**) which defines the menu items
in the application. Create a new menu item as shown below (You can add it
right after the dashboard menu item).

    new AppMenuItem("PhoneBook", null, "flaticon-book", "/app/main/phonebook")

**PhoneBook** is the menu name (will localize below), **null** is a placeholder for the 
permission name (will set it later), **flaticon-book** is just an
arbitrary icon class (from [this
set](http://keenthemes.com/metronic/preview/?page=components/icons/flaticon&demo=default))
and **app/main/phonebook** is the Angular route.

If you run the application, you can see a new menu item in the left
menu.  But it will not work (it redirect to default route) if you click to
the menu item, since we have not defined the Angular route yet.

#### Localize Menu Item Display Name

Localization strings are defined in **XML** files in the **.Core** project
(in server side) as shown below:

<img src="images/localization-files-4.png" alt="Localization files" class="img-thumbnail" />

Open PhoneBookDemo.xml (the **default**, **English** localization
dictionary) and add the following line:

    <text name="PhoneBook">Phone Book</text>

If we don't define the "PhoneBook" value for other localization
dictionaries, the default value is shown in all languages. For example, we
can also define it for Turkish in the PhoneBookDmo-tr.xml file:

    <text name="PhoneBook">Telefon Rehberi</text>

Note: Any change in server side code (including changes to localization text)
requires a recycle of the server application. If
you don't need to debug, we suggest you use Ctrl+F5 for a faster startup. It will re-build and recycle the application.

#### Angular Route

Angular has a powerful URL routing system. ASP.NET Zero has defined
routes in a few places (for modularity, see [dev
guide](Development-Guide-Angular.md)). We want to add the phone book page
to the main module. So, open **src\\app\\main\\main-routing.module.ts**
(in the client side) and add a new route just below the dashboard route:

    { path: 'phonebook', component: PhoneBookComponent }

We will get an error since we haven't defined the PhoneBookComponent yet. Also,
we are ignoring permission for now (will implement later).

### Creating the PhoneBookComponent

Create a **phonebook** folder inside **src/app/main** folder and add a
new typescript file (**phonebook.component.ts**) in the phonebook folder
as shown below:

    import { Component, Injector } from '@angular/core';
    import { AppComponentBase } from '@shared/common/app-component-base';
    import { appModuleAnimation } from '@shared/animations/routerTransition';

    @Component({
        templateUrl: './phonebook.component.html',
        animations: [appModuleAnimation()]
    })

    export class PhoneBookComponent extends AppComponentBase {
        constructor(
            injector: Injector
        ) {
            super(injector);
        }
    }

We inherited from **AppComponentBase** which provides some common
functions and fields (like localization and access control) for us. It's
not required, but makes our job easier. Now, we can go back to
**main-routing.module.ts** and add an import statement for the newly
created PhoneBookComponent class:

    import { PhoneBookComponent } from './phonebook/phonebook.component';

As we declared in **phonebook.component.ts**, we should create a
**phonebook.component.html** view in the same phonebook folder:

    <div [@routerTransition]>
        <div class="m-subheader ">
            <div class="d-flex align-items-center">
                <div class="mr-auto col-sm-6">
                    <h3 class="m-subheader__title m-subheader__title--separator">
                        <span>{{l("PhoneBook")}}</span>
                    </h3>
                </div>
            </div>
        </div>
        <div class="m-content">
            <div class="m-portlet m-portlet--mobile">
                <div class="m-portlet__body">
                    <p>PHONE BOOK CONTENT COMES HERE!</p>
                </div>
            </div>
        </div>
    </div>

**l** (lower case 'L') function comes from **AppComponentBase** and can be used
to easily localize texts. The **\[@routerTransition\]** attribute is
required for page transition animation.

And finally, Angular requires to relate every component to a **module**.
So, we are making the following changes to **main.module.ts**:

    import { NgModule } from '@angular/core';
    import { CommonModule } from '@angular/common';
    import { FormsModule } from '@angular/forms';

    import { DashboardComponent } from './dashboard/dashboard.component';
    import { PhoneBookComponent } from './phonebook/phonebook.component';

    import { ModalModule, TabsModule, TooltipModule } from 'ngx-bootstrap';
    import { AppCommonModule } from '@app/shared/common/app-common.module';
    import { UtilsModule } from '@shared/utils/utils.module';
    import { MainRoutingModule } from './main-routing.module';

    @NgModule({
        imports: [
            CommonModule,
            FormsModule,
            ModalModule,
            TabsModule,
            TooltipModule,
            AppCommonModule,
            UtilsModule,
            MainRoutingModule
        ],
        declarations: [
          DashboardComponent,
          PhoneBookComponent
        ]
    })
    export class MainModule { }

We Added an **import** statement and added PhoneBookComponent to the
**declarations** array. Now, we can refresh the page to see the new
page we added:

<img src="images/phonebook-empty-ng2.png" alt="Phonebook empty" class="img-thumbnail" style="width:100.0%" />

Note: Angular-cli automatically re-compiles and refreshes the page when
any changes are made to any file in the application.

### Creating Person Entity

We define entities in the **.Core** (domain) project (in server side). We
can define a **Person** entity (mapped to **PbPersons** table in
database) to represent a person in the phone book as shown below (I created
it in a new folder/namespace named PhoneBook):

    using System.ComponentModel.DataAnnotations;
    using System.ComponentModel.DataAnnotations.Schema;
    using Abp.Domain.Entities.Auditing;

    namespace Acme.PhoneBookDemo.PhoneBook
    {
        [Table("PbPersons")]
        public class Person : FullAuditedEntity
        {
            public const int MaxNameLength = 32;
            public const int MaxSurnameLength = 32;
            public const int MaxEmailAddressLength = 255;

            [Required]
            [MaxLength(MaxNameLength)]
            public virtual string Name { get; set; }

            [Required]
            [MaxLength(MaxSurnameLength)]
            public virtual string Surname { get; set; }

            [MaxLength(MaxEmailAddressLength)]
            public virtual string EmailAddress { get; set; }
        }
    }

Person's **primary key** type is **int** (as default). It inherits
**FullAuditedEntity** which contains **creation**, **modification** and
**deletion** audit properties. It also has **soft-delete**. With **soft-delete**, when we delete
a person, it's not deleted in the database but marked as deleted (see
[entity](https://aspnetboilerplate.com/Pages/Documents/Entities) and
[data
filters](https://aspnetboilerplate.com/Pages/Documents/Data-Filters) for more information). We created consts for
**MaxLength** properties. This is a good practice since we will use the same
values later.

We need to add a DbSet property for Person entity to **PhoneBookDemoDbContext**
class defined in the **.EntityFrameworkCore** project.

    public class PhoneBookDemoDbContext : AbpZeroDbContext<Tenant, Role, User>
    {
        public virtual DbSet<Person> Persons { get; set; }

        //...other code
    }

### Database Migrations

We use **EntityFramework Code-First migrations** to migrate database
schemas. Since we added **Person entity**, our DbContext model has
changed. So, we should create a **new migration** to create the new
table in the database.

Open **Package Manager Console**, run the **Add-Migration
"Added\_Persons\_Table"** command as shown below:

<img src="images/phonebook-migrations-core-3.png" alt="Entity Framework Code First Migration" class="img-thumbnail" />

This command will add a **migration class** named
"**Added\_Persons\_Table**" as shown below:

    public partial class Added_Persons_Table : Migration
    {
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.CreateTable(
                name: "PbPersons",
                columns: table => new
                {
                    Id = table.Column(nullable: false)
                        .Annotation("SqlServer:ValueGenerationStrategy", SqlServerValueGenerationStrategy.IdentityColumn),
                    CreationTime = table.Column(nullable: false),
                    CreatorUserId = table.Column(nullable: true),
                    DeleterUserId = table.Column(nullable: true),
                    DeletionTime = table.Column(nullable: true),
                    EmailAddress = table.Column(maxLength: 255, nullable: true),
                    IsDeleted = table.Column(nullable: false),
                    LastModificationTime = table.Column(nullable: true),
                    LastModifierUserId = table.Column(nullable: true),
                    Name = table.Column(maxLength: 32, nullable: false),
                    Surname = table.Column(maxLength: 32, nullable: false)
                },
                constraints: table =>
                {
                    table.PrimaryKey("PK_PbPersons", x => x.Id);
                });
        }

        protected override void Down(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.DropTable(
                name: "PbPersons");
        }
    }

We don't have to know too much about the format or rules of this file, but 
it's suggested to have a basic understanding of migrations. In the same
Package Manager Console, we run the **Update-Database** command in order
to apply the new migration to the database. After updating, we can see that the
**PbPersons table** is added to database.

<img src="images/phonebook-tables-spa.png" alt="Phonebook tables" class="img-thumbnail" />

For now, the new table is empty. In ASP.NET Zero, there are some classes to
fill initial data for users and settings:

<img src="images/aspnet-core-ef-seed-1.png" alt="Seed folders" class="img-thumbnail" />

So, we can add a separate class to populate some people in the database as
shown below:

    namespace Acme.PhoneBookDemo.Migrations.Seed.Host
    {
        public class InitialPeopleCreator
        {
            private readonly PhoneBookDemoDbContext _context;

            public InitialPeopleCreator(PhoneBookDemoDbContext context)
            {
                _context = context;
            }

            public void Create()
            {
                var douglas = _context.Persons.FirstOrDefault(p => p.EmailAddress == "douglas.adams@fortytwo.com");
                if (douglas == null)
                {
                    _context.Persons.Add(
                        new Person
                        {
                            Name = "Douglas",
                            Surname = "Adams",
                            EmailAddress = "douglas.adams@fortytwo.com"
                        });
                }

                var asimov = _context.Persons.FirstOrDefault(p => p.EmailAddress == "isaac.asimov@foundation.org");
                if (asimov == null)
                {
                    _context.Persons.Add(
                        new Person
                        {
                            Name = "Isaac",
                            Surname = "Asimov",
                            EmailAddress = "isaac.asimov@foundation.org"
                        });
                }
            }
        }
    }

This type of default data is good since we can also use this data in
**unit tests**. However, we should be careful about seed data since this
code will always be executed in each **PostInitialize** of your
PhoneBookEntityFrameworkCoreModule. This class (InitialPeopleCreator) is
created and called in the **InitialHostDbBuilder** class. This is not so
important, just good code organization (see source codes).

    public class InitialHostDbBuilder
    {
        //existing codes...

        public void Create()
        {
            //existing code...
            new InitialPeopleCreator(_context).Create();

            _context.SaveChanges();
        }
    }

Now when we run our project again, it runs the seed process and adds two people to PbPersons
table:

<img src="images/phonebook-persons-table-initial-data.png" alt="Persons initial data" class="img-thumbnail" width="720" height="50" />

### Creating Person Application Service

An Application Service is used by clients (presentation layer) to
perform operations (use cases) in the application.

Application service interfaces and DTOs are located in the
**.Application.Shared** project. We are creating an application service to
get people from the server. So, first we need to create an **interface** to
define the person application service (while this interface is optional,
we suggest you to create it):

    using Abp.Application.Services;
    using Abp.Application.Services.Dto;

    namespace Acme.PhoneBookDemo.PhoneBook
    {
        public interface IPersonAppService : IApplicationService
        {
            ListResultDto<PersonListDto> GetPeople(GetPeopleInput input);
        }
    }

An application service method gets/returns **DTO**s. **ListResultDto**
is a pre-build helper DTO to return a list of another DTO.
**GetPeopleInput** is a DTO to pass request parameters to the **GetPeople**
method. So, GetPeopleIntput and PersonListDto are defined as shown
below:

    public class GetPeopleInput
    {
        public string Filter { get; set; }
    }

    public class PersonListDto : FullAuditedEntityDto
    {
        public string Name { get; set; }

        public string Surname { get; set; }

        public string EmailAddress { get; set; }
    }

**CustomDtoMapper.cs** is used to create the mapping from **Person** to
**PersonListDto**. **FullAuditedEntityDto** is inherited to
implement audit properties automatically. See [application
service](https://aspnetboilerplate.com/Pages/Documents/Application-Services)
and
[DTO](https://aspnetboilerplate.com/Pages/Documents/Data-Transfer-Objects) 
for more information. We need to add the following mappings:

    ...
    // PhoneBook (we will comment out other lines when the new DTOs are added)
    configuration.CreateMap<Person, PersonListDto>();
    //configuration.CreateMap<AddPhoneInput, Phone>();
    //configuration.CreateMap<CreatePersonInput, Person>();
    //configuration.CreateMap<Person, GetPersonForEditOutput>();
    //configuration.CreateMap<Phone, PhoneInPersonListDto>();

After defining the interface, we can implement it as shown below: (in
**.Application** project)

    public class PersonAppService : PhoneBookDemoAppServiceBase, IPersonAppService
    {
        private readonly IRepository<Person> _personRepository;

        public PersonAppService(IRepository<Person> personRepository)
        {
            _personRepository = personRepository;
        }

        public ListResultDto<PersonListDto> GetPeople(GetPeopleInput input)
        {
            var people = _personRepository
                .GetAll()
                .WhereIf(
                    !input.Filter.IsNullOrEmpty(),
                    p => p.Name.Contains(input.Filter) ||
                         p.Surname.Contains(input.Filter) ||
                         p.EmailAddress.Contains(input.Filter)
                )
                .OrderBy(p => p.Name)
                .ThenBy(p => p.Surname)
                .ToList();

            return new ListResultDto<PersonListDto>(ObjectMapper.Map<List<PersonListDto>>(people));
        }
    }

We're injecting **person repository** (it's automatically created by
ABP) and using it to filter and get people from the database.

**WhereIf** is an extension method here (defined in Abp.Linq.Extensions
namespace). It performs a where condition, only if the filter is not null or
empty. **IsNullOrEmpty** is also an extension method (defined in
Abp.Extensions namespace). ABP has many similar shortcut extension
methods. **ObjectMapper.Map** method automatically converts a list of
Person entities to a list of PersonListDto objects using
configurations defined in **CustomDtoMapper.cs** in the **.Application** project.

#### Connection & Transaction Management

We don't manually open database connection or start/commit transactions. 
They are automatically done with ABP framework's Unit Of Work
system. See [UOW
documentation](https://aspnetboilerplate.com/Pages/Documents/Unit-Of-Work)
for more information.

#### Exception Handling

We don't handle exceptions manually (using a try-catch block). Because
the ABP framework automatically handles all exceptions in the web layer and
returns appropriate error messages to the client, it then processes errors
on the client and shows the error information to the user. See
[exception handling
document](https://aspnetboilerplate.com/Pages/Documents/Handling-Exceptions)
for more information.

### Creating Unit Tests For PersonAppService

You can skip this section if you have no interest in **automated
testing**.

By writing a unit test, we can test the **PersonAppService.GetPeople** method
without creating a user interface. We will write the unit test in the .**Tests**
project in the solution.

#### MultiTenancy In Tests

Since we disabled multitenancy, we should also disable it for unit tests.
Open the **PhoneBookDemoConsts** class in the Acme.PhoneBook.Core project
and set "**MultiTenancyEnabled**" to false. After a rebuild, run unit
tests and you will see that some tests are skipped (those are related to
multitenancy).

Let's create our first test to verify getting people without any filter:

    using Acme.PhoneBookDemo.People;
    using Acme.PhoneBookDemo.People.Dtos;
    using Shouldly;
    using Xunit;

    namespace Acme.PhoneBookDemo.Tests.People
    {
        public class PersonAppService_Tests : AppTestBase
        {
            private readonly IPersonAppService _personAppService;

            public PersonAppService_Tests()
            {
                _personAppService = Resolve<IPersonAppService>();
            }

            [Fact]
            public void Should_Get_All_People_Without_Any_Filter()
            {
                //Act
                var persons = _personAppService.GetPeople(new GetPeopleInput());

                //Assert
                persons.Items.Count.ShouldBe(2);
            }
        }
    }

We derived the test class from **AppTestBase**. The AppTestBase class
initializes the system, creates an in-memory fake database, seeds
initial data (that we created before) to the database and logs into to
application as admin. So, this is actually an **integration test** since
it tests all server-side code ranging from entity framework mapping to
application services, validation and authorization.

In the constructor, we get (resolve) an **IPersonAppService** from the 
**dependency injection** container. It creates the **PersonAppService**
class with all of the dependencies. Then we can use it in test methods.

Since we're using [xUnit](http://xunit.github.io/), we add the **Fact**
attribute to each test method. In the test method, we called the
**GetPeople** method and checked to see if there are **two people** in the
returned list as we know that there are 2 people in **initial**
database.

Let's run the **all unit tests** in Test Explorer and see if it works:

<img src="images/phone-book-unit-test-success-2.png" alt="xUnit unit test success" class="img-thumbnail" />

As you see, it worked **successfully**. Now, we know that
PersonAppService works properly without any filter. Let's add a new unit
test to get filtered people:

    [Fact]
    public void Should_Get_People_With_Filter()
    {
        //Act
        var persons = _personAppService.GetPeople(
            new GetPeopleInput
            {
                Filter = "adams"
            });

        //Assert
        persons.Items.Count.ShouldBe(1);
        persons.Items[0].Name.ShouldBe("Douglas");
        persons.Items[0].Surname.ShouldBe("Adams");
    }

Again, since we know the initial database, we can check the returned results
easily. Here, the initial test data is important. When we change the initial
data, our tests may fail even if our services are correct. So, it's
better to write unit tests independent of the initial data as much as
possible. We could check incoming data to see if every person contains
"adams" in his/her name, surname or email. Thus, if we add new people to
the initial data, our unit tests will still work.

There are many techniques to unit testing. I kept it simple here. But
the ASP.NET Zero template makes it very easy to write unit and integration
tests by base classes and pre-build test codes. 

### Using GetPeople Method From Angular Component

Now, we can switch to the client side and use the GetPeople method to show a
list of people in the UI.

#### Service Proxy Generation

First, run (prefer Ctrl+F5 because it is faster) the server side application
(.Web.Host project). Then run the **nswag/refresh.bat** file on the client
side to re-generate service proxy classes (they are used to call server
side service methods).

Since we added a new service, we should add it to
**src/shared/service-proxies/service-proxy.module.ts**. Just open it and
add **ApiServiceProxies.PersonServiceProxy** to the providers array.
This step is only required when we add a new service. If we change an
existing service, it's not needed.

#### Angular-Cli Watcher

Sometimes angular-cli cannot understand file changes. When this happens,
stop it and re-run the **npm start** command.

#### PhoneBookComponent Typescript Class

Change **phonebook.component.ts** as shown below:

    import { Component, Injector, OnInit } from '@angular/core';
    import { AppComponentBase } from '@shared/common/app-component-base';
    import { appModuleAnimation } from '@shared/animations/routerTransition';
    import { PersonServiceProxy, PersonListDto, ListResultDtoOfPersonListDto } from '@shared/service-proxies/service-proxies';

    @Component({
        templateUrl: './phonebook.component.html',
        animations: [appModuleAnimation()]
    })
    export class PhoneBookComponent extends AppComponentBase implements OnInit {

        people: PersonListDto[] = [];
        filter: string = '';

        constructor(
            injector: Injector,
            private _personService: PersonServiceProxy
        ) {
            super(injector);
        }

        ngOnInit(): void {
            this.getPeople();
        }

        getPeople(): void {
            this._personService.getPeople(this.filter).subscribe((result) => {
                this.people = result.items;
            });
        }
    }

We inject **PersonServiceProxy**, call its **getPeople** method and
**subscribe** to get the result. We do this in the **ngOnInit** function
(defined in Angular's **OnInit** interface). Then we assign the returned items to
the **people** class member.

#### Rendering People In Angular View

Now, we can use the people member for the view,
**phonebook.component.html**:

    <div [@routerTransition]>
        <div class="m-subheader ">
            <div class="d-flex align-items-center">
                <div class="mr-auto col-sm-6">
                    <h3 class="m-subheader__title m-subheader__title--separator">
                        <span>{{l("PhoneBook")}}</span>
                    </h3>
                </div>
            </div>
        </div>
        <div class="m-content">
            <div class="m-portlet m-portlet--mobile">
                <div class="m-portlet__body">

                    <h3>{{l("AllPeople")}}</h3>
                    <div class="m-widget1">
                        <div class="m-widget1__item" *ngFor="let person of people">
                            <div class="row m-row--no-padding align-items-center">
                                <div class="col">
                                    <h3 class="m-widget1__title">{{person.name + ' ' + person.surname}}</h3>
                                    <span class="m-widget1__desc">{{person.emailAddress}}</span>
                                </div>
                            </div>
                        </div>
                    </div>

                </div>
            </div>
        </div>
    </div>

We simply used the **ngFor** directive to loop and render the people data. Here is
the result:

<img src="images/phonebook-people-view-2.png" alt="Phonebook peoples" class="img-thumbnail" />

We have successfully retrieved the list of people from the database to the page.

#### About Showing Tabular Data

We normally use a javascript based rich table/grid library to show
tabular data, instead of manually rendering data using HTML. For example,
we used the [jTable](http://jtable.org/) library to show users on the Users
page of ASP.NET Zero. Consider using such components since they make things
much easier and provide a much better user experience.

We did not use a table component here, because we wanted to show the basics of
Angular instead of going into the details of a 3rd party library.

### Creating a New Person

Next step is to create a modal to add a new item to the phone book.

#### Add a CreatePerson Method to PersonAppService

We first define the **CreatePerson** method in the **IPersonAppService**
interface:

    Task CreatePerson(CreatePersonInput input);

Then we create the **CreatePersonInput** DTO that defines the parameters of the
method:

    public class CreatePersonInput
    {
        [Required]
        [MaxLength(PersonConsts.MaxNameLength)]
        public string Name { get; set; }

        [Required]
        [MaxLength(PersonConsts.MaxSurnameLength)]
        public string Surname { get; set; }

        [EmailAddress]
        [MaxLength(PersonConsts.MaxEmailAddressLength)]
        public string EmailAddress { get; set; }
    }

**CreatePersonInput** is mapped to the **Person** entity (comment out
the related line in CustomDtoMapper.cs and we will use mapping below).
All properties are decorated with **data annotation attributes**
to provide automatic
**[validation](https://aspnetboilerplate.com/Pages/Documents/Validating-Data-Transfer-Objects)**.
Notice that we use the same consts defined in **PersonConsts.cs** in
the **.Core.Shared** project for **MaxLength** properties. After adding this
class, you can remove consts from the **Person** entity and use this new
consts class.

    public class PersonConsts
    {
        public const int MaxNameLength = 32;
        public const int MaxSurnameLength = 32;
        public const int MaxEmailAddressLength = 255;
    }

Here is the implementation of the CreatePerson method:

    public async Task CreatePerson(CreatePersonInput input)
    {
        var person = ObjectMapper.Map<Person>(input);
        await _personRepository.InsertAsync(person);
    }

A Person entity is created by mapping the provided input and then inserting into
the database. We used the **async/await** pattern here. All methods in ASP.NET
Zero startup project are **async**. It's advised to use async/await
wherever possible.

#### Test CreatePerson Method

You can skip this section if you are not interested in **automated
testing**.

We can create a unit test method to test the CreatePerson method as shown
below:

    [Fact]
    public async Task Should_Create_Person_With_Valid_Arguments()
    {
        //Act
        await _personAppService.CreatePerson(
            new CreatePersonInput
            {
                Name = "John",
                Surname = "Nash",
                EmailAddress = "john.nash@abeautifulmind.com"
            });

        //Assert
        UsingDbContext(
            context =>
            {
                var john = context.Persons.FirstOrDefault(p => p.EmailAddress == "john.nash@abeautifulmind.com");
                john.ShouldNotBe(null);
                john.Name.ShouldBe("John");
            });
    }

The test method is also written using the **async/await** pattern since
the calling method is async.  Next, we called the CreatePerson method, 
then checked if the provided person is in the database. The **UsingDbContext** 
method is a helper method of **AppTestBase** class (which we inherited 
this unit test class from). It's used to easily get a reference to DbContext 
and use it directly to perform database operations.

This method successfully works since all required fields are supplied.
Let's try to create a test for **invalid arguments**:

    [Fact]
    public async Task Should_Not_Create_Person_With_Invalid_Arguments()
    {
        //Act and Assert
        await Assert.ThrowsAsync<AbpValidationException>(
            async () =>
                    {
                        await _personAppService.CreatePerson(
                            new CreatePersonInput
                            {
                                Name = "John"
                            });
                    });
    }

We did not set the **Surname** property of CreatePersonInput despite it being
**required**. So, it throws the **AbpValidationException** automatically.
Also, we cannot send null to the CreatePerson method since the validation
system also checks it. This test calls CreatePerson with invalid
arguments and asserts that it throws AbpValidationException. See
[validation
document](https://aspnetboilerplate.com/Pages/Documents/Validating-Data-Transfer-Objects)
for more information.

#### Creating a Modal

We will create a Bootstrap Modal to create a new person. ASP.NET Zero
uses [ngx-bootstrap](https://github.com/valor-software/ngx-bootstrap)
library to create modals (you can use another library, but we will use
it in this sample). Final modal should look like this.

<img src="images/phonebook-create-person-dialog1.png" alt="Create Person Dialog" class="img-thumbnail" />

First of all, we should use **nswag/refresh.bat** to re-generate
service-proxies. This will generate the code that is needed to call the
PersonAppService.**CreatePerson** method from the client. Notice that
you should rebuild & run the server side application before
re-generating the proxy scripts.

We will start by creating a new component, named
**create-person-modal.component.ts** in the client side phonebook folder:

    import { Component, ViewChild, Injector, ElementRef, Output, EventEmitter } from '@angular/core';
    import { ModalDirective } from 'ngx-bootstrap';
    import { PersonServiceProxy, CreatePersonInput } from '@shared/service-proxies/service-proxies';
    import { AppComponentBase } from '@shared/common/app-component-base';

    @Component({
        selector: 'createPersonModal',
        templateUrl: './create-person-modal.component.html'
    })
    export class CreatePersonModalComponent extends AppComponentBase {

        @Output() modalSave: EventEmitter<any> = new EventEmitter<any>();

        @ViewChild('modal') modal: ModalDirective;
        @ViewChild('nameInput') nameInput: ElementRef;

        person: CreatePersonInput = new CreatePersonInput();

        active: boolean = false;
        saving: boolean = false;

        constructor(
            injector: Injector,
            private _personService: PersonServiceProxy
        ) {
            super(injector);
        }

        show(): void {
            this.active = true;
            this.person = new CreatePersonInput();
            this.modal.show();
        }

        onShown(): void {
            $(this.nameInput.nativeElement).focus();
        }

        save(): void {
            this.saving = true;
            this._personService.createPerson(this.person)
                .finally(() => this.saving = false)
                .subscribe(() => {
                    this.notify.info(this.l('SavedSuccessfully'));
                    this.close();
                    this.modalSave.emit(this.person);
                });
        }

        close(): void {
            this.modal.hide();
            this.active = false;
        }
    }

Let me explain some parts of this class:

-   It has a selector, **createPersonModal**, which will be used like
    an HTML element in the person list page.
-   It extends **AppComponentBase** to take advantage of helper methods (defines
    this.l and this.notify in this sample).
-   Defines an event, **modalSave**, which is triggered when we
    successfully save the modal. Thus, the main page will be informed
    and it can reload the person list.
-   Declares two **ViewChild** members (**modal** and **nameInput**) to
    access some elements in the view.
-   Injects **PersonServiceProxy** to call server side method while
    creating the person.
-   It sets the focus on the **name** input when the modal is shown.

The code is simple and easy to understand except a small hack: an active
flag is used to reset validation for the Angular view (explained in
angular's
[documentation](https://angular.io/docs/ts/latest/cookbook/form-validation.html)).

As declared in the component, we are creating the
**create-person-modal.component.html** file in the same folder as shown
below:

    <div bsModal #modal="bs-modal" (onShown)="onShown()" class="modal fade" tabindex="-1" role="dialog" aria-labelledby="modal" aria-hidden="true" [config]="{backdrop: 'static'}">
        <div class="modal-dialog modal-lg">
            <div class="modal-content">
                <form *ngIf="active" #personForm="ngForm" novalidate (ngSubmit)="save()">
                    <div class="modal-header">
                        <h4 class="modal-title">
                            <span>{{l("CreateNewPerson")}}</span>
                        </h4>
                        <button type="button" class="close" (click)="close()" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <div class="form-group">
                            <label>{{l("Name")}}</label>
                            <input #nameInput class="form-control" type="text" name="name" [(ngModel)]="person.name" required maxlength="32">
                        </div>
                        <div class="form-group">
                            <label>{{l("Surname")}}</label>
                            <input class="form-control" type="email" name="surname" [(ngModel)]="person.surname" required maxlength="32">
                        </div>
                        <div class="form-group">
                            <label>{{l("EmailAddress")}}</label>
                            <input class="form-control" type="email" name="emailAddress" [(ngModel)]="person.emailAddress" required maxlength="255" pattern="^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{1,})+$">
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button [disabled]="saving" type="button" class="btn btn-secondary" (click)="close()">{{l("Cancel")}}</button>
                        <button type="submit" class="btn btn-primary" [disabled]="!personForm.form.valid" [buttonBusy]="saving" [busyText]="l('SavingWithThreeDot')"><i class="fa fa-save"></i> <span>{{l("Save")}}</span></button>
                    </div>
                </form>
            </div>
        </div>
    </div>

Most of this code is the same for all modals. The important part is how
we bound the model to the view using the ngModel directive. Like all
components, Angular requires it to be related to a module. We should add it to the
**declarations** array of **main.module.ts** as shown below:

    ...previous imports
    import { CreatePersonModalComponent } from './phonebook/create-person-modal.component';

    @NgModule({
        imports: [
            ...existing module imports...
        ],
        declarations: [
            DashboardComponent,
            PhoneBookComponent,
            CreatePersonModalComponent
        ]
    })
    export class MainModule { }

We need to add a "Create new person" button to the 'people list page' to
open the modal when clicking on the button. To do that, we made the
following changes in **phonebook.component.html**:

    <div [@routerTransition]>
        <div class="m-subheader ">
            <div class="d-flex align-items-center">
                <div class="mr-auto col-sm-6">
                    <h3 class="m-subheader__title m-subheader__title--separator">
                        <span>{{l("PhoneBook")}}</span>
                    </h3>
                </div>
                <div class="col-sm-6 text-right">
                    <button class="btn btn-primary" (click)="createPersonModal.show()"><i class="fa fa-plus"></i> {{l("CreateNewPerson")}}</button>
                </div>
            </div>
        </div>
        <div class="m-content">
            <div class="m-portlet m-portlet--mobile">
                <div class="m-portlet__body">
                    <h3>{{l("AllPeople")}}</h3>
                    <div class="m-widget1">
                        <div class="m-widget1__item" *ngFor="let person of people">
                            <div class="row m-row--no-padding align-items-center">
                                <div class="col">
                                    <h3 class="m-widget1__title">{{person.name + ' ' + person.surname}}</h3>
                                    <span class="m-widget1__desc">{{person.emailAddress}}</span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <createPersonModal #createPersonModal (modalSave)="getPeople()"></createPersonModal>
    </div>

Made some minor changes in the view; Added a **button** to open the
modal and added the **createPersonModal** component as an HTML tag
(which matches to the selector in **create-person-modal.component.ts**).

### Authorization For Phone Book

At this point, anyone can use the phone book page since no authorization has been
defined. To fix this, we will define two permissions:

-   A permission to **use the phone book page**.
-   A permission to **create a new person** (which naturally is a child permission
    of first one).

#### Permission for Entering Phone Book Page

##### Define the permission

Go to **AppAuthorizationProvider** class in the server side and add a
new permission as shown below (you can add just below the dashboard
permission):

    pages.CreateChildPermission(AppPermissions.Pages_Tenant_PhoneBook, L("PhoneBook"), multiTenancySides: MultiTenancySides.Tenant);

A permission should have a unique name. We define permission names as
constant strings in **AppPermissions** class. It's a simple constant
string:

    public const string Pages_Tenant_PhoneBook = "Pages.Tenant.PhoneBook";

The unique name of this permission is "**Pages.Tenant.PhoneBook**". While
you can set any string (as long as it's unique), it's suggested to use
this convention. A permission can also have a localizable display name.
In this case we chose "*PhoneBook**" (See "Adding a New Page" section for
more about localization, since it's very similar). Lastly, we set this 
permission as a **tenant** level permission.

##### Add AbpAuthorize attribute

The **AbpAuthorize** attribute can be used at the **class** or **method** 
level to protect an application service or service method from
unauthorized requests. Since all server side code is located in
PersonAppService class, we can declare a class level attribute as shown
below:

    [AbpAuthorize(AppPermissions.Pages_Tenant_PhoneBook)]
    public class PersonAppService : PhoneBookAppServiceBase, IPersonAppService
    {
        //...
    }

Now, let's try to enter the Phone Book page by clicking the menu item:

<img src="images/phonebook-permission-error.png" alt="Permission error" class="img-thumbnail" width="505" height="412" />

We get an error message. This exception is thrown when any method of
PersonAppService is called without the required permission.

##### Guard Angular Route

Even though we got a permission exception and the server did not send the data, 
we can still enter the page. To prevent this from happening, open **main-routing.module.ts** 
and make the following changes to the route definition:

    { path: 'phonebook', component: PhoneBookComponent, data: { permission: 'Pages.Tenant.PhoneBook' } }

The **AuthRouteGuard** class automatically checks the route permission data and
prevents users from entering the view if the specified permission is not granted.
Try to click the Phone Book menu now!

##### Hide Unauthorized Menu Item

While the user can not enter to the page, the menu item is still there! We
should also **hide** the Phone book **menu item**. This is easy to do.  Open
**app-navigation-service.ts** and change the PhoneBook menu definition
as shown below:

    new AppMenuItem("PhoneBook", 'Pages.Tenant.PhoneBook', "flaticon-book", "/app/main/phonebook")

##### Grant permission

So, how do we access the page now? Simple, go to the **Role Management** page and 
edit the **admin** role:

<img src="images/role-permissions-with-phonebook1.png" alt="Role permissions" class="img-thumbnail" />

We see that a **new permission** named "**Phone book**" has been added 
to **permissions** tab. So, we check it and then save the role. After
saving, we need to **refresh** the whole page to refresh permissions for
the current user. We could also grant this permission for a specific
user (see [development guide document](Development-Guide.md) for
details about roles and users).

Now, we can access the Phone book page again.

#### Permission for Create New Person

While a permission for a page is useful and probably always needed, we
may want to define additional permissions to perform some **specific
actions** on a page, like creating a new person.

##### Define the Permission

Defining a permission is similar (in the AppAuthorizationProvider
class):

    var phoneBook = pages.CreateChildPermission(AppPermissions.Pages_Tenant_PhoneBook, L("PhoneBook"), multiTenancySides: MultiTenancySides.Tenant);
    phoneBook.CreateChildPermission(AppPermissions.Pages_Tenant_PhoneBook_CreatePerson, L("CreateNewPerson"), multiTenancySides: MultiTenancySides.Tenant);

The first permission above is the parent permission. In the second line, we are creating
a child permission of the parent. Remember to create a constant in the
AppPermissions class:

    public const string Pages_Tenant_PhoneBook_CreatePerson = "Pages.Tenant.PhoneBook.CreatePerson";

##### Add AbpAuthorize Attribute

This time, we're applying the **AbpAuthorize** attribute only for the
**CreatePerson** method:

    [AbpAuthorize(AppPermissions.Pages_Tenant_PhoneBook_CreatePerson)]
    public async Task CreatePerson(CreatePersonInput input)
    {
        //...
    }

##### Hide Unauthorized Button

If we run the application and try to create a person, we get an
authorization error after clicking the save button. But, shouldn't we
**completely hide the Create New Person button** if the user doesn't have 
the correct permission. It's very simple.  Just make the foolowing changes:

Open the **phonebook.component.html** view and use the **isGranted** condition 
as shown below:

    <button *ngIf="isGranted('Pages.Tenant.PhoneBook.CreatePerson')" class="btn btn-primary" (click)="createPersonModal.show()"><i class="fa fa-plus"></i> {{l("CreateNewPerson")}}</button>

In this way, the "Create New Person" button does not rendered in the UI
and the user cannot see the button.

##### Grant permission

To see the button again, we can go to role or user manager and grant
related permission as shown below:

<img src="images/user-permissions-phonebook1.png" alt="User specific permissions" class="img-thumbnail" />

As shown above, **Create new person** permission is a child permission
of the **Phone book**. Remember to refresh page to get permissions
updated.

### Deleting a Person

Let's add a delete button in people list as shown below:

<img src="images/phonebook-people-delete-button1.png" alt="Delete person" class="img-thumbnail" />

We're starting from UI in this case.

#### View

We're changing **phonebook.component.html** view to add a delete button
(related part is shown here):

    ...
    <h3>{{l("AllPeople")}}</h3>
    <div class="m-widget1">
        <div class="m-widget1__item" *ngFor="let person of people">
            <div class="row m-row--no-padding align-items-center">
                <div class="col">
                    <h3 class="m-widget1__title">{{person.name + ' ' + person.surname}}</h3>
                    <span class="m-widget1__desc">{{person.emailAddress}}</span>
                </div>
                <div class="col m--align-right">
                    <button id="deletePerson" (click)="deletePerson(person)" title="{{l('Delete')}}" class="btn btn-outline-danger m-btn m-btn--icon m-btn--icon-only m-btn--pill" href="javascript:;">
                        <i class="fa fa-times"></i>
                    </button>
                </div>
            </div>
        </div>
    </div>
    ...

We simply added a button which calls **deletePerson** method (will be
defined) when it's clicked. You can define a permission for 'deleting
person' as we did for 'creating person' above.

#### Style

We're using a **[LESS](http://lesscss.org/)** style here to take the
button right. Created a file named **phonebook.component.less** (in
phonebook folder) and added following lines:

    .m-widget1__item{
        button#deletePerson {
            /* styles */
        }
    }

And adding the style to the **phonebook.component.ts** Component
declaration:

    @Component({
        templateUrl: './phonebook.component.html',
        styleUrls: ['./phonebook.component.less'],
        animations: [appModuleAnimation()]
    })

Now, we can now the buttons, but they don't work since we haven't
defined the deletePerson method yet.

#### Application Service

Let's leave the client side and add a DeletePerson method to the server
side. We are adding it to the service interface,**IPersonAppService:**:

    Task DeletePerson(EntityDto input);

**EntityDto** is a shortcut of ABP if we only get an id value.
Implementation (in **PersonAppService**) is very simple:

    [AbpAuthorize(AppPermissions.Pages_Tenant_PhoneBook_DeletePerson)]
    public async Task DeletePerson(EntityDto input)
    {
        await _personRepository.DeleteAsync(input.Id);
    }

#### Service Proxy Generation

Since we changed server side services, we should re-generate the client
side service proxies via NSwag. Make server side running and use
refresh.bat as we did before.

#### Component Script

Now, we can add **deletePerson** method to **phonebook.component.ts**:

    deletePerson(person: PersonListDto): void {
        this.message.confirm(
            this.l('AreYouSureToDeleteThePerson', person.name),
            isConfirmed => {
                if (isConfirmed) {
                    this._personService.deletePerson(person.id).subscribe(() => {
                        this.notify.info(this.l('SuccessfullyDeleted'));
                        _.remove(this.people, person);
                    });
                }
            }
        );
    } 

It first shows a confirmation message when we click the delete button:

<img src="images/confirmation-delete-person1.png" alt="Confirmation message" class="img-thumbnail" />

If we click Yes, it simply calls **deletePerson** method of
**PersonAppService** and shows a
**[notification](https://aspnetboilerplate.com/Pages/Documents/Javascript-API/Notification)**
if operation succeed. Also, removes the person from the person array
using [lodash](https://lodash.com/) library. We also added an import
statement before the @Component declaration:

    import * as _ from 'lodash';

### Filtering people

Now, we will implement **search** functionality of **GetPeople** method.
UI is shown below:

<img src="images/search-people1.png" alt="Searching people" class="img-thumbnail" />

We added a search input to **phonebook.component.html** view (showing
the related part of the code):

    <h3>{{l("AllPeople")}} ({{people.length}})</h3>
    <form autocomplete="off">
        <div class="m-form m-form--label-align-right">
            <div class="row align-items-center m--margin-bottom-10">
                <div class="col-xl-12">
                    <div class="form-group m-form__group align-items-center">
                        <div class="input-group">
                            <input [(ngModel)]="filter" name="filterText" autoFocus class="form-control m-input" [placeholder]="l('SearchWithThreeDot')" type="text">
                            <span class="input-group-btn">
                                <button (click)="getPeople()" class="btn btn-primary" type="submit"><i class="flaticon-search-1"></i></button>
                            </span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </form>

    <div class="m-widget1">
        <div class="m-widget1__item" *ngFor="let person of people">
            <div class="row m-row--no-padding align-items-center">
                <div class="col">
                    <h3 class="m-widget1__title">{{person.name + ' ' + person.surname}}</h3>
                    <span class="m-widget1__desc">{{person.emailAddress}}</span>
                </div>
                <div class="col m--align-right">
                    <button id="deletePerson" (click)="deletePerson(person)" title="{{l('Delete')}}" class="btn btn-outline-danger m-btn m-btn--icon m-btn--icon-only m-btn--pill" href="javascript:;">
                        <i class="fa fa-times"></i>
                    </button>
                </div>
            </div>
        </div>
    </div>

We also added currently filtered people count (people.length) in the
header. Since we have already defined and used the filter property in
**phonebook.component.ts** and implemented in the server side, this new
code immediately works.

### Extending the Application: Adding Phone Numbers

Until now, we did not even mention about phone numbers. It's time to
extend our domain to support **multiple phone numbers** for a person.

### Creating Phone Entity

Let's start by creating a new Entity, **Phone** in **.Core** project:

    [Table("PbPhones")]
    public class Phone : CreationAuditedEntity<long>
    {
        public const int MaxNumberLength = 16;

        [ForeignKey("PersonId")]
        public virtual Person Person { get; set; }
        public virtual int PersonId { get; set; }

        [Required]
        public virtual PhoneType Type { get; set; }

        [Required]
        [MaxLength(MaxNumberLength)]
        public virtual string Number { get; set; }
    }

Phone entities are stored in **PbPhones** table. Its primary key is
**long** and it inherits creation auditing properties. It has a reference
to **Person** entity which owns the phone number.

We added a **Phones** collection to the People:

    [Table("PbPersons")]
    public class Person : FullAuditedEntity
    {
        //...other properties

        public virtual ICollection<Phone> Phones { get; set; }
    }

We have a **PhoneType** enum as shown below: (in **.Application.Shared**
project)

    public enum PhoneType : byte
    {
        Mobile,
        Home,
        Business
    }

Lastly, we're also adding a DbSet property for Phone to our DbContext:

    public virtual DbSet<Phone> Phones { get; set; }

### Adding Database Migration

Our entity model has changed, so we need to add a new migration. Run
this command in the .EntityFramework project's directory:

<img src="images/phonebook-migrations-core-4.png" alt="Entity Framework Migration" class="img-thumbnail" />

This will create a new code based migration file to create **PbPhones**
table:

    public partial class Added_Phone : Migration
    {
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.CreateTable(
                name: "PbPhones",
                columns: table => new
                {
                    Id = table.Column<long>(nullable: false)
                        .Annotation("SqlServer:ValueGenerationStrategy", SqlServerValueGenerationStrategy.IdentityColumn),
                    CreationTime = table.Column<DateTime>(nullable: false),
                    CreatorUserId = table.Column<long>(nullable: true),
                    Number = table.Column<string>(maxLength: 16, nullable: false),
                    PersonId = table.Column<int>(nullable: false),
                    Type = table.Column<byte>(nullable: false)
                },
                constraints: table =>
                {
                    table.PrimaryKey("PK_PbPhones", x => x.Id);
                    table.ForeignKey(
                        name: "FK_PbPhones_PbPersons_PersonId",
                        column: x => x.PersonId,
                        principalTable: "PbPersons",
                        principalColumn: "Id",
                        onDelete: ReferentialAction.Cascade);
                });

            migrationBuilder.CreateIndex(
                name: "IX_PbPhones_PersonId",
                table: "PbPhones",
                column: "PersonId");
        }

        protected override void Down(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.DropTable(
                name: "PbPhones");
        }
    }

Before updating database, we can go to database
**InitialPeopleCreator**, rename it to **InitialPeopleAndPhoneCreator**
and add example **phone numbers** for example people (We renamed
InitialPeopleCreator.cs to InitialPeopleAndPhoneCreator.cs):

    public class InitialPeopleAndPhoneCreator
    {
        //...

        public void Create()
        {
            var douglas = _context.Persons.FirstOrDefault(p => p.EmailAddress == "douglas.adams@fortytwo.com");
            if (douglas == null)
            {
                _context.Persons.Add(
                    new Person
                    {
                        Name = "Douglas",
                        Surname = "Adams",
                        EmailAddress = "douglas.adams@fortytwo.com",
                        Phones = new List<Phone>
                                    {
                                        new Phone {Type = PhoneType.Home, Number = "1112242"},
                                        new Phone {Type = PhoneType.Mobile, Number = "2223342"}
                                    }
                    });
            }

            var asimov = _context.Persons.FirstOrDefault(p => p.EmailAddress == "isaac.asimov@foundation.org");
            if (asimov == null)
            {
                _context.Persons.Add(
                    new Person
                    {
                        Name = "Isaac",
                        Surname = "Asimov",
                        EmailAddress = "isaac.asimov@foundation.org",
                        Phones = new List<Phone>
                                    {
                                        new Phone {Type = PhoneType.Home, Number = "8889977"}
                                    }
                    });
            }
        }
    }

We added two phone numbers to Douglas, one phone number to Isaac. But if
we run our application now, phones are not inserted since this seed code
checks if person exists, and does not insert if it already exists.
Since we haven't deployed yet, we can delete database
(or remove entries from people table) and re-create it.

Now, we are running our application to re-create database and seed it.
You can check database to see **PbPhones** table and rows.

### Changing GetPeople Method

We're changing **PersonAppService.GetPeople** method to **include**
phone numbers of people into return value.

First, we're changing **PersonListDto** to contain a list of phones:

    public class PersonListDto : FullAuditedEntityDto
    {
        public string Name { get; set; }

        public string Surname { get; set; }

        public string EmailAddress { get; set; }

        public Collection<PhoneInPersonListDto> Phones { get; set; }
    }

    public class PhoneInPersonListDto : CreationAuditedEntityDto<long>
    {
        public PhoneType Type { get; set; }

        public string Number { get; set; }
    }

So, added also a DTO to transfer phone numbers and mapped from Phone
entity. Now, we can change GetPeople method to get Phones from database:

    public ListResultDto<PersonListDto> GetPeople(GetPeopleInput input)
    {
        var persons = _personRepository
            .GetAll()
            .Include(p => p.Phones)
            .WhereIf(
                !input.Filter.IsNullOrEmpty(),
                p => p.Name.Contains(input.Filter) ||
                        p.Surname.Contains(input.Filter) ||
                        p.EmailAddress.Contains(input.Filter)
            )
            .OrderBy(p => p.Name)
            .ThenBy(p => p.Surname)
            .ToList();

        return new ListResultDto<PersonListDto>(ObjectMapper.Map<List<PersonListDto>>(persons));
    }

We only added **Include** extension method to the query. Rest of the
codes remains same. Furthermore, it would work without adding this, but
much slower (since it will lazy load phone numbers for every person
separately).

### AddPhone and DeletePhone Methods

We are adding two more methods to IPersonAppService interface as shown
below:

    Task DeletePhone(EntityDto<long> input);
    Task<PhoneInPersonListDto> AddPhone(AddPhoneInput input);

We could create a new, separated IPhoneAppService. It's your choice.
But, we can consider Person as an aggregate and add phone related
methods here. AddPhoneInput DTO is shown below:

    public class AddPhoneInput
    {
        [Range(1, int.MaxValue)]
        public int PersonId { get; set; }

        [Required]
        public PhoneType Type { get; set; }

        [Required]
        [MaxLength(PhoneConsts.MaxNumberLength)]
        public string Number { get; set; }
    }

We used PhoneConsts.MaxNumberLength for Number field. You should create
this consts in **.Core.Shared**.

    public class PhoneConsts
    {
        public const int MaxNumberLength = 16;
    }

Now, we can implement these methods:

    public async Task DeletePhone(EntityDto<long> input)
    {
        await _phoneRepository.DeleteAsync(input.Id);
    }

    public async Task<PhoneInPersonListDto> AddPhone(AddPhoneInput input)
    {
        var person = _personRepository.Get(input.PersonId);
        await _personRepository.EnsureCollectionLoadedAsync(person, p => p.Phones);

        var phone = ObjectMapper.Map<Phone>(input);
        person.Phones.Add(phone);

        //Get auto increment Id of the new Phone by saving to database
        await CurrentUnitOfWork.SaveChangesAsync();

        return ObjectMapper.Map<PhoneInPersonListDto>(phone);
    }

(Note: We injected **IRepository&lt;Phone, long&gt;** in the constructor
and set to \_phoneRepository field, as similar to \_personRepository)

**DeletePhone** method is simple. It only deletes phone with given id.

**AddPhone** method **gets** the person from database and add new phone
to person.Phones collection. Then is **save changes**. Saving changes
causes inserting new added phone to database and get its **Id**.
Because, we are returning a DTO that contains newly created phone
informations including Id. So, it should be assigned before mapping in
the last line. (Notice that; normally it's not needed to call
CurrentUnitOfWork.SaveChangesAsync. It's automatically called at the end
of the method. We called it in the method since we need to save entity
and get its Id immediately. See [UOW
document](https://aspnetboilerplate.com/Pages/Documents/Unit-Of-Work#DocAutoSaveChanges)
for more.)

There may be different approaches for AddPhone method. You can directly
work with a **phone repository** to insert new phone. They all have
different pros and cons. It's your choice.

### Edit Mode For Phone Numbers

Final UI is shown below:

<img src="images/phone-book-edit-mode1.png" alt="Phone book edit mode" class="img-thumbnail" />

When we click the **edit icon** for a person, its row is expanded and
all phone numbers are shown. Then we can delete any phone by clicking
the icon at left. We can add a new phone from the inputs at last line.

#### View

Changes in view are shown below:

    <div class="m-widget1__item" *ngFor="let person of people" [ngClass]="{'bg-secondary m--padding-10': person===editingPerson}">
        <div class="row m-row--no-padding align-items-center">
            <div class="col">
                <h3 class="m-widget1__title">{{person.name + ' ' + person.surname}}</h3>
                <span class="m-widget1__desc">{{person.emailAddress}}</span>
            </div>
            <div class="col m--align-right">
                <button (click)="editPerson(person)" title="{{l('Edit')}}" class="btn btn-outline-primary m-btn m-btn--icon m-btn--icon-only m-btn--pill">
                    <i class="fa fa-pencil"></i>
                </button>
                <button id="deletePerson" (click)="deletePerson(person)" title="{{l('Delete')}}" class="btn btn-outline-danger m-btn m-btn--icon m-btn--icon-only m-btn--pill" href="javascript:;">
                    <i class="fa fa-times"></i>
                </button>
            </div>
        </div>
        <div class="row">
            <div class="col-sm-12 m--margin-top-20" *ngIf="person===editingPerson">
                <table class="table table-hover">
                    <thead>
                        <tr>
                            <th style="width:10%"></th>
                            <th style="width:15%">{{l("Type")}}</th>
                            <th style="width:75%">{{l("PhoneNumber")}}</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr *ngFor="let phone of person.phones">
                            <td>
                                <button (click)="deletePhone(phone, person)" class="btn btn-outline-danger m-btn m-btn--icon m-btn--icon-only m-btn--pill">
                                    <i class="fa fa-times"></i>
                                </button>
                            </td>
                            <td>{{getPhoneTypeAsString(phone.type)}}</td>
                            <td>{{phone.number}}</td>
                        </tr>
                        <tr>
                            <td>
                                <button (click)="savePhone()" class="btn btn-sm btn-success">
                                    <i class="fa fa-floppy-o"></i>
                                </button>
                            </td>
                            <td>
                                <select name="Type" [(ngModel)]="newPhone.type"class="form-control">
                                    <option value="0">{{l("PhoneType_Mobile")}}</option>
                                    <option value="1">{{l("PhoneType_Home")}}</option>
                                    <option value="2">{{l("PhoneType_Business")}}</option>
                                </select>
                            </td>
                            <td><input type="text" name="number" [(ngModel)]="newPhone.number" class="form-control" /></td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

We added an edit button for each person. Then added a table for each
person that shows phones of the related person and allows adding a new
phone. Phones table is only shown if we click the edit button.

#### PhoneBook Component Class

Before changing PhoneBookComponent, we should re-generate
service-proxies using nswag as did above. And finally we can change
PhoneBookComponent as shown below:

    import { Component, Injector, OnInit } from '@angular/core';
    import { AppComponentBase } from '@shared/common/app-component-base';
    import { appModuleAnimation } from '@shared/animations/routerTransition';
    import { PersonServiceProxy, PersonListDto, ListResultDtoOfPersonListDto, PhoneInPersonListDto, AddPhoneInput, AddPhoneInputType, PhoneInPersonListDtoType } from '@shared/service-proxies/service-proxies';

    import * as _ from 'lodash';

    @Component({
        templateUrl: './phonebook.component.html',
        styleUrls: ['./phonebook.component.less'],
        animations: [appModuleAnimation()]
    })
    export class PhoneBookComponent extends AppComponentBase implements OnInit {

        people: PersonListDto[] = [];
        filter: string = '';

        editingPerson: PersonListDto = null;
        newPhone: AddPhoneInput = null;

        constructor(
            injector: Injector,
            private _personService: PersonServiceProxy
        ) {
            super(injector);
        }

        ngOnInit(): void {
            this.getPeople();
        }

        getPeople(): void {
            this._personService.getPeople(this.filter).subscribe((result) => {
                this.people = result.items;
            });
        }

        deletePerson(person: PersonListDto): void {
            this.message.confirm(
                this.l('AreYouSureToDeleteThePerson', person.name),
                isConfirmed => {
                    if (isConfirmed) {
                        this._personService.deletePerson(person.id).subscribe(() => {
                            this.notify.info(this.l('SuccessfullyDeleted'));
                            _.remove(this.people, person);
                        });
                    }
                }
            );
        }

        editPerson(person: PersonListDto): void {
            if (person === this.editingPerson) {
                this.editingPerson = null;
            } else {
                this.editingPerson = person;

                this.newPhone = new AddPhoneInput();
                this.newPhone.type = AddPhoneInputType._0;
                this.newPhone.personId = person.id;
            }
        };

        getPhoneTypeAsString(phoneType: PhoneInPersonListDtoType): string {
            switch (phoneType) {
                case PhoneInPersonListDtoType._0:
                    return this.l('PhoneType_Mobile');
                case PhoneInPersonListDtoType._1:
                    return this.l('PhoneType_Home');
                case PhoneInPersonListDtoType._2:
                    return this.l('PhoneType_Business');
                default:
                    return '?';
            }
        };

        deletePhone(phone, person): void {
            this._personService.deletePhone(phone.id).subscribe(() => {
                this.notify.success(this.l('SuccessfullyDeleted'));
                _.remove(person.phones, phone);
            });
        };

        savePhone(): void {
            if (!this.newPhone.number) {
                this.message.warn('Please enter a number!');
                return;
            }

            this._personService.addPhone(this.newPhone).subscribe(result => {
                this.editingPerson.phones.push(result);
                this.newPhone.number = '';

                this.notify.success(this.l('SavedSuccessfully'));
            });
        };
    }

### Edit Mode For People

Now we want to edit name, surname and e-mail of people:

<img src="images/edit-person-core1.png" alt="Edit Person" class="img-thumbnail" />  

First of all, we create the necessary DTOs to transfer people's id, name,
surname and e-mail. Then create the functions in PersonAppService for
editing people:  

    [AbpAuthorize(AppPermissions.Pages_Tenant_PhoneBook_EditPerson)]
    public async Task<GetPersonForEditOutput> GetPersonForEdit(GetPersonForEditInput input)
    {
        var person = await _personRepository.GetAsync(input.Id);
        return ObjectMapper.Map<GetPersonForEditOutput>(person);
    }

    [AbpAuthorize(AppPermissions.Pages_Tenant_PhoneBook_EditPerson)]
    public async Task EditPerson(EditPersonInput input)
    {
        var person = await _personRepository.GetAsync(input.Id);
        person.Name = input.Name;
        person.Surname = input.Surname;
        person.EmailAddress = input.EmailAddress;
        await _personRepository.UpdateAsync(person);
    }

#### View

Create edit-person-modal.component.html:

    <div bsModal #modal="bs-modal" (onShown)="onShown()" class="modal fade" tabindex="-1" role="dialog" aria-labelledby="modal" aria-hidden="true" [config]="{backdrop: 'static'}">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <form *ngIf="active" #personForm="ngForm" novalidate (ngSubmit)="save()">
            <div class="modal-header">
                <h4 class="modal-title">
                <span>{{l("EditPerson")}}</span>
              </h4>
              <button type="button" class="close" (click)="close()" aria-label="Close">
                <span aria-hidden="true">&times;</span>
              </button>          
            </div>
            <div class="modal-body">

              <div class="form-group">
                <label>{{l("Name")}}</label>
                <input #nameInput class="form-control" type="text" name="name" [(ngModel)]="person.name" required maxlength="32">            
              </div>

              <div class="form-group">
                <label>{{l("Surname")}}</label>
                <input class="form-control" type="email" name="surname" [(ngModel)]="person.surname" required maxlength="32">
              </div>

              <div class="form-group">
              <label>{{l("EmailAddress")}}</label>
                <input class="form-control" type="email" name="emailAddress" [(ngModel)]="person.emailAddress" required maxlength="255" pattern="^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{1,})+$">                        
              </div>

            </div>
            <div class="modal-footer">
              <button [disabled]="saving" type="button" class="btn btn-secondary" (click)="close()">{{l("Cancel")}}</button>
              <button type="submit" class="btn btn-primary" [disabled]="!personForm.form.valid" [buttonBusy]="saving" [busyText]="l('SavingWithThreeDot')"><i class="fa fa-save"></i> <span>{{l("Save")}}</span></button>
            </div>
          </form>
        </div>
      </div>
    </div>

Add those lines to **phonebook.component.html:**:

            <button (click)="editPerson(person)" title="{{l('Edit')}}" class="btn btn-outline-primary m-btn m-btn--icon m-btn--icon-only m-btn--pill">
                <i class="fa fa-plus"></i>
            </button>
            <button *ngIf="isGranted('Pages.Tenant.PhoneBook.EditPerson')" (click)="editPersonModal.show(person.id)" title="{{l('EditPerson')}}" class="btn btn-outline-success m-btn m-btn--icon m-btn--icon-only m-btn--pill">
                <i class="fa fa-pencil"></i>
            </button>
           <button id="deletePerson" (click)="deletePerson(person)" title="{{l('Delete')}}" class="btn btn-outline-danger m-btn m-btn--icon m-btn--icon-only m-btn--pill" href="javascript:;">
                <i class="fa fa-times"></i>
            </button>
                    .
                    .
                    .
                    .
                    .
    <createPersonModal #createPersonModal(modalSave)="getPeople()"></createPersonModal>
    <editPersonModal #editPersonModal (modalSave)="getPeople()"></editPersonModal>

#### Controller

Create edit-person-modal.component.ts:

    import { Component, ViewChild, Injector, ElementRef, Output, EventEmitter } from '@angular/core';
    import { ModalDirective } from 'ngx-bootstrap';
    import { PersonServiceProxy, EditPersonInput } from '@shared/service-proxies/service-proxies';
    import { AppComponentBase } from '@shared/common/app-component-base';

    @Component({
      selector: 'editPersonModal',
      templateUrl: './edit-person-modal.component.html'
    })
    export class EditPersonModalComponent extends AppComponentBase {

      @Output() modalSave: EventEmitter<any> = new EventEmitter<any>();

      @ViewChild('modal') modal: ModalDirective;
      @ViewChild('nameInput') nameInput: ElementRef;

      person: EditPersonInput = new EditPersonInput();

      active: boolean = false;
      saving: boolean = false;

      constructor(
        injector: Injector,
        private _personService: PersonServiceProxy
      ) {
        super(injector);
      }

      show(personId): void {
        this.active = true;
        this._personService.getPersonForEdit(personId).subscribe((result)=> {
          this.person = result;
          this.modal.show();
        });

      }

      onShown(): void {
       // $(this.nameInput.nativeElement).focus();
      }

      save(): void {
        this.saving = true;
        this._personService.editPerson(this.person)
          .subscribe(() => {
            this.notify.info(this.l('SavedSuccessfully'));
            this.close();
            this.modalSave.emit(this.person);
          });
        this.saving = false;
      }

      close(): void {
        this.modal.hide();
        this.active = false;
      }
    }

Add those lines to **main.module.ts:**:

        import { EditPersonModalComponent } from './phonebook/edit-person-modal.component';
    .
    .
    .
    .
    .
    .
        declarations: [
          DashboardComponent,
          PhoneBookComponent,
          CreatePersonModalComponent,
          EditPersonModalComponent
        ]
    .
    .


### Multi Tenancy

We have built a fully functional application until here. Now, we will
see how to convert it to a multi-tenant application easily. Logout from
the application before any change.

#### Enable Multi Tenancy

We disabled multi-tenancy at the beginning of this document. Now,
re-enabling it in **PhoneBookDemoConsts** class:

    public const bool MultiTenancyEnabled = true;

#### Make Entities Multi Tenant

In a multi-tenant application, a tenant's entities should be isolated by
other tenants. For this example project, every tenant should have own
phone book with isolated people and phone numbers.

When we implement IMustHaveTenant interface, ABP automatically [filters
data](https://aspnetboilerplate.com/Pages/Documents/Data-Filters) based
on current Tenant, while retrieving entities from database. So, we
should declare that Person entity must have a tenant using
**IMustHaveTenant** interface:

    public class Person : FullAuditedEntity, IMustHaveTenant
    {
        public virtual int TenantId { get; set; }

        //...other properties
    }

We may want to add IMustHaveTenant interface to also Phone entity. This
is needed if we directly use phone repository to get phones. In this
sample project, it's not needed.

Since entities have changed, we should create a new database migration:

    Add-Migration "Implemented_IMustHaveTenant_For_Person"

This command creates a new code-first database migration. The migration
class adds an annotation this is needed for automatic filtering. We
don't have to know what it is since it's done automatically. It also
adds a **TenantId** column to PbPersons table as shown below:

    migrationBuilder.AddColumn<int>(name: "TenantId",table: "PbPersons",nullable: false,defaultValue: 1);

I added **defaultValue as 1** to AddColumn options. Thus, current people
are automatically assigned to **default tenant** (default tenant's id is
always 1).

Now, we can update the database again:

    Update-Database

#### Run The Multi Tenant Application

**It's finished**! We can test the application. Run the project,
**login** as the **host admin** (click Change link and clear tenancy
name) shown below:

<img src="images/login-as-host-3.png" alt="Login as host" class="img-thumbnail" style="width:100.0%" />

After login, we see the **tenant list** which only contains a
**default** tenant. We can create a new tenant:

<img src="images/create-tenant-6.png" alt="Creating tenant" class="img-thumbnail" />

I created a new tenant named **trio**. Now, tenant list has two tenants:

<img src="images/tenant-management-phonebook-2.png" alt="Tenant management page" class="img-thumbnail" style="width:100.0%" />

 I can **logout** and login as **trio** tenant **admin** (Change current
tenant to trio):

<img src="images/login-as-trio1.png" alt="Login as tenant admin" class="img-thumbnail" style="width:100.0%" />

After login, we see that phone book is empty:

<img src="images/phonebook-empty1.png" alt="Empty phonebook of new tenant" class="img-thumbnail" />

It's empty because trio tenant has a completely isolated people list.
You can add people here, logout and login as different tenants (you can
login as default tenant for example). You will see that each tenant has
an isolated phone book and can not see other's people.

### Code Generation

Using new **ASP.NET Zero Power Tools**, you can speed up your development.

See [documentation](https://aspnetzero.com/Documents/Development-Guide-Rad-Tool) to learn how to use it.

### Conclusion

In this document, we built a complete example that covers most parts of
the ASP.NET Zero system. We hope that it will help you to build your own
application.

We intentionally used different approaches for similar tasks to show you
different styles of development. ASP.NET Zero provides an architecture
but does not restrict you. You can make your own style development.

#### Source Code

You should [purchase](/Prices) ASP.NET Zero in order to get **source
code**. After purchasing, you can get the sample project from private
Github repository: <https://github.com/aspnetzero/aspnet-zero-samples>
