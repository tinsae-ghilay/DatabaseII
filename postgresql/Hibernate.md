# Hibernate

### Relationships

##### OneToOne
for one to one, we add an object on one side
```java
@Entity
@Table(name="") // table name
public class Person{
    @ID
    @GeneratedValue(strategy=Generation.AUTO)
    private int id
    @Column(nullable=false)
    private String firstName;
    @Column(nullable=false)
    private String lastName;
    @OneToOne
    @JoinColumn ( name = " AddressId " ) // we add FK column name
    private Address address;

    //getter and setter
}

@Entity
@Table(name="") // table name
public class Address{
    @ID
    @GeneratedValue(strategy=Generation.AUTO)
    private int id
    @Column
    private String street;
    @Comlumn(name="House_number")
    private String housNr;

    //getter and setter
}
```

#### embedable
No key required and Embeded entity must not be an Entity. so no `@Entity` required
Object data is embeded in the Table (Embeder) itself
```java
@Entity
@Table(name="") // table name
public class Person{
    @ID
    @GeneratedValue(strategy=Generation.AUTO)
    private int id
    @Column(nullable=false)
    private String firstName;
    @Column(nullable=false)
    private String lastName;
    @Embeded
    private Address address;

    //getter and setter
}


public class Address{
    private String street;
    private String housNr;

    //getter and setter
}
```
#### One to many (1:N)
Relation represented using  `@OneToMany` on 1 side and `@ManyToOne` on the many side<br>
E.g  `[Address](N)----<IN>----(1)[City]`

```java
```java
@Entity
@Table(name="") // table name
public class City{
    @ID
    @GeneratedValue(strategy=Generation.AUTO)
    private int id
    @Column(nullable=false)
    private String name;
    @Column(nullable=false)
    private String lastName;
    // on the one side we declare a List of the Many
    @OneToMany(mappedBy="City") // map to object in Address class
    private List<Address> addresses;

    //getter and setter
}

@Entity
@Table(name="") // table name
public class Address{
    @ID
    @GeneratedValue(strategy=Generation.AUTO)
    private int id
    @Column
    private String street;
    @Comlumn(name="House_number")
    private String housNr;
    @ManyToOne // on the many side, we declare the object and here is where the FK is
    @JoinColumn(name = "city_id")
    City city

    //getter and setter
}
```

#### Many To Many

For a Many To Many relationship we add a List of each other on both sides. But on one side we do `MappedBy` and on the other side we add 
```java
@JoinTable(name = "listName"),{
    JoinColumns ={@joinColumn (name ="column_name_here")},
    InverseJoinColumn{@JoinColumn(name = "column_name_on_another")}
    
}
```
***Example*** For a Relationship with `[Person](N)---<Visits>---(M)[Course]`

```java
@Entity
class Person {
    @Id @Generated @Column ( name = " personId " )
    private Long id ;
    @ManyToMany ( mappedBy = " visitors " )
    private List < Course > courses ;
}


@Entity
class Course {
    @Id @Generated @Column ( name = " courseId " )
    private Long id ;
    @ManyToMany
    @JoinTable ( name = " PersonCourses " ,
    joinColumns ={ @JoinColumn ( name = " personId " ) } ,
    inverseJoinColumns ={ @JoinColumn ( name = " courseId " ) })
    private List < Person > visitors ;
}
```
