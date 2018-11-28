### hibernate相关知识

#### 注解

##### @Formula

hibernate插入数据时能够对列进行操作，无须在java中操作

```java
@Entity(name = "Account")
public static class Account {

	@Id
	private Long id;

	private Double credit;

	private Double rate;

	@Formula(value = "credit * rate")
	private Double interest;

	//Getters and setters omitted for brevity

}
```



##### @UpdateTimestamp

对数据进行更新的时候，hibernate自动更新当前系统时间，无需java操作

```java
	@Column(name = "updated_on")
	@UpdateTimestamp
	private Date updatedOn;
```



##### @CreationTimestamp

新增一条记录时，系统自动将当前`jvm`时间插入到记录中

```java
	@Column(name = "creat_on")
	@CreationTimestamp
	private Date CreatOn
```



##### @ColumnTransformer

从数据库读取或插入数据时，对列的数据进行相关转换

```java
	@ColumnTransformer(
		read = "money / 100",
		write = "? * 100"
	)
	private MonetaryAmount wallet;
```



##### @Where

查询时添加查询对结果进行过滤

```java
@Entity(name = "Account")
@Where( clause = "active = true" )
public static class Account {

	@Id
	private Long id;

	@ManyToOne
	private Client client;

	@Column(name = "account_type")
	@Enumerated(EnumType.STRING)
	private AccountType type;

	private Double amount;

	private Double rate;

	private boolean active;

	//Getters and setters omitted for brevity

}
```



##### @whereJionTable

和where一致，对查询的结果进行筛选

```java
@Entity(name = "Book")
public static class Book {

	@Id
	private Long id;

	private String title;

	private String author;

	@ManyToMany
	@JoinTable(
		name = "Book_Reader",
		joinColumns = @JoinColumn(name = "book_id"),
		inverseJoinColumns = @JoinColumn(name = "reader_id")
	)
	@WhereJoinTable( clause = "created_on > DATEADD( 'DAY', -7, CURRENT_TIMESTAMP() )")
	private List<Reader> currentWeekReaders = new ArrayList<>( );

	//Getters and setters omitted for brevity

}

@Entity(name = "Reader")
public static class Reader {

	@Id
	private Long id;

	private String name;

	//Getters and setters omitted for brevity

}
create table Book (
    id bigint not null,
    author varchar(255),
    title varchar(255),
    primary key (id)
)

create table Book_Reader (
    book_id bigint not null,
    reader_id bigint not null
)

create table Reader (
    id bigint not null,
    name varchar(255),
    primary key (id)
)

alter table Book_Reader
    add constraint FKsscixgaa5f8lphs9bjdtpf9g
    foreign key (reader_id)
    references Reader

alter table Book_Reader
    add constraint FKoyrwu9tnwlukd1616qhck21ra
    foreign key (book_id)
    references Book

alter table Book_Reader
    add created_on timestamp
    default current_timestamp
```



##### @Filter

对查询到的结果进行筛选,允许传递属性值

```java
@Entity(name = "Account")
@FilterDef(
    name="activeAccount",
    parameters = @ParamDef(
        name="active",
        type="boolean"
    )
)
@Filter(
    name="activeAccount",
    condition="active_status = :active"
)
public static class Account {

    @Id
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Client client;

    @Column(name = "account_type")
    @Enumerated(EnumType.STRING)
    private AccountType type;

    private Double amount;

    private Double rate;

    @Column(name = "active_status")
    private boolean active;

    //Getters and setters omitted for brevity
}
```



##### @ManyToOne

* 多对一：在多的一方中书写，joinColumn在多的一方生成一个列，为外部键。不会生成额外表

```java
@Entity(name = "Person")
public static class Person {

	@Id
	@GeneratedValue
	private Long id;

	//Getters and setters are omitted for brevity

}

@Entity(name = "Phone")
public static class Phone {

	@Id
	@GeneratedValue
	private Long id;

	@Column(name = "`number`")
	private String number;

	@ManyToOne
	@JoinColumn(name = "person_id",
			foreignKey = @ForeignKey(name = "PERSON_ID_FK")
	)
	private Person person;

	//Getters and setters are omitted for brevity

}
CREATE TABLE Person (
    id BIGINT NOT NULL ,
    PRIMARY KEY ( id )
)

CREATE TABLE Phone (
    id BIGINT NOT NULL ,
    number VARCHAR(255) ,
    person_id BIGINT ,
    PRIMARY KEY ( id )
 )

ALTER TABLE Phone
ADD CONSTRAINT PERSON_ID_FK
FOREIGN KEY (person_id) REFERENCES Person
```



##### @OneToMany

* 建立一张中间表来保存主键和外键关联

```java
@Entity(name = "Person")
public static class Person {

	@Id
	@GeneratedValue
	private Long id;

	@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
	private List<Phone> phones = new ArrayList<>();

	//Getters and setters are omitted for brevity

}

@Entity(name = "Phone")
public static class Phone {

	@Id
	@GeneratedValue
	private Long id;

	@Column(name = "`number`")
	private String number;

	//Getters and setters are omitted for brevity

}
CREATE TABLE Person (
    id BIGINT NOT NULL ,
    PRIMARY KEY ( id )
)

CREATE TABLE Person_Phone (
    Person_id BIGINT NOT NULL ,
    phones_id BIGINT NOT NULL
)

CREATE TABLE Phone (
    id BIGINT NOT NULL ,
    number VARCHAR(255) ,
    PRIMARY KEY ( id )
)

ALTER TABLE Person_Phone
ADD CONSTRAINT UK_9uhc5itwc9h5gcng944pcaslf
UNIQUE (phones_id)

ALTER TABLE Person_Phone
ADD CONSTRAINT FKr38us2n8g5p9rj0b494sd3391
FOREIGN KEY (phones_id) REFERENCES Phone

ALTER TABLE Person_Phone
ADD CONSTRAINT FK2ex4e4p7w1cj310kg2woisjl2
FOREIGN KEY (Person_id) REFERENCES Person
```


* 双向联系    -----  表和表之间通过mappedBy，进行联系

```java
@Entity(name = "Person")
public static class Person {

	@Id
	@GeneratedValue
	private Long id;

	@OneToMany(mappedBy = "person", cascade = CascadeType.ALL, orphanRemoval = true)
	private List<Phone> phones = new ArrayList<>();

	//Getters and setters are omitted for brevity

	public void addPhone(Phone phone) {
		phones.add( phone );
		phone.setPerson( this );
	}

	public void removePhone(Phone phone) {
		phones.remove( phone );
		phone.setPerson( null );
	}
}

@Entity(name = "Phone")
public static class Phone {

	@Id
	@GeneratedValue
	private Long id;

	@NaturalId
	@Column(name = "`number`", unique = true)
	private String number;

	@ManyToOne
	private Person person;

	//Getters and setters are omitted for brevity

	@Override
	public boolean equals(Object o) {
		if ( this == o ) {
			return true;
		}
		if ( o == null || getClass() != o.getClass() ) {
			return false;
		}
		Phone phone = (Phone) o;
		return Objects.equals( number, phone.number );
	}

	@Override
	public int hashCode() {
		return Objects.hash( number );
	}
}
CREATE TABLE Person (
    id BIGINT NOT NULL ,
    PRIMARY KEY ( id )
)

CREATE TABLE Phone (
    id BIGINT NOT NULL ,
    number VARCHAR(255) ,
    person_id BIGINT ,
    PRIMARY KEY ( id )
)

ALTER TABLE Phone
ADD CONSTRAINT UK_l329ab0g4c1t78onljnxmbnp6
UNIQUE (number)

ALTER TABLE Phone
ADD CONSTRAINT FKmw13yfsjypiiq0i1osdkaeqpg
FOREIGN KEY (person_id) REFERENCES Person
```



##### @OneToOne

* 在写OneToOne的类中生成一个新的列，用来映射两张表之间的关系 --- 单向联系

```java
@Entity(name = "Phone")
public static class Phone {

	@Id
	@GeneratedValue
	private Long id;

	@Column(name = "`number`")
	private String number;

	@OneToOne
	@JoinColumn(name = "details_id")
	private PhoneDetails details;

	//Getters and setters are omitted for brevity

}

@Entity(name = "PhoneDetails")
public static class PhoneDetails {

	@Id
	@GeneratedValue
	private Long id;

	private String provider;

	private String technology;

	//Getters and setters are omitted for brevity

}
CREATE TABLE Phone (
    id BIGINT NOT NULL ,
    number VARCHAR(255) ,
    details_id BIGINT ,
    PRIMARY KEY ( id )
)

CREATE TABLE PhoneDetails (
    id BIGINT NOT NULL ,
    provider VARCHAR(255) ,
    technology VARCHAR(255) ,
    PRIMARY KEY ( id )
)

ALTER TABLE Phone
ADD CONSTRAINT FKnoj7cj83ppfqbnvqqa5kolub7
FOREIGN KEY (details_id) REFERENCES PhoneDetails
```

* 双向联系

```java
@Entity(name = "Phone")
public static class Phone {

	@Id
	@GeneratedValue
	private Long id;

	@Column(name = "`number`")
	private String number;

	@OneToOne(
		mappedBy = "phone",
		cascade = CascadeType.ALL,
		orphanRemoval = true,
		fetch = FetchType.LAZY
	)
	private PhoneDetails details;

	//Getters and setters are omitted for brevity

	public void addDetails(PhoneDetails details) {
		details.setPhone( this );
		this.details = details;
	}

	public void removeDetails() {
		if ( details != null ) {
			details.setPhone( null );
			this.details = null;
		}
	}
}

@Entity(name = "PhoneDetails")
public static class PhoneDetails {

	@Id
	@GeneratedValue
	private Long id;

	private String provider;

	private String technology;

	@OneToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "phone_id")
	private Phone phone;

	//Getters and setters are omitted for brevity

}
CREATE TABLE Phone (
    id BIGINT NOT NULL ,
    number VARCHAR(255) ,
    PRIMARY KEY ( id )
)

CREATE TABLE PhoneDetails (
    id BIGINT NOT NULL ,
    provider VARCHAR(255) ,
    technology VARCHAR(255) ,
    phone_id BIGINT ,
    PRIMARY KEY ( id )
)

ALTER TABLE PhoneDetails
ADD CONSTRAINT FKeotuev8ja8v0sdh29dynqj05p
FOREIGN KEY (phone_id) REFERENCES Phone
```



```java
@Entity(name = "Person")
public static class Person implements Serializable {

	@Id
	@GeneratedValue
	private Long id;

	@NaturalId
	private String registrationNumber;

	@OneToMany(
		mappedBy = "person",
		cascade = CascadeType.ALL,
		orphanRemoval = true
	)
	private List<PersonAddress> addresses = new ArrayList<>();

	//Getters and setters are omitted for brevity

	public void addAddress(Address address) {
		PersonAddress personAddress = new PersonAddress( this, address );
		addresses.add( personAddress );
		address.getOwners().add( personAddress );
	}

	public void removeAddress(Address address) {
		PersonAddress personAddress = new PersonAddress( this, address );
		address.getOwners().remove( personAddress );
		addresses.remove( personAddress );
		personAddress.setPerson( null );
		personAddress.setAddress( null );
	}

	@Override
	public boolean equals(Object o) {
		if ( this == o ) {
			return true;
		}
		if ( o == null || getClass() != o.getClass() ) {
			return false;
		}
		Person person = (Person) o;
		return Objects.equals( registrationNumber, person.registrationNumber );
	}

	@Override
	public int hashCode() {
		return Objects.hash( registrationNumber );
	}
}

@Entity(name = "PersonAddress")
public static class PersonAddress implements Serializable {

	@Id
	@ManyToOne
	private Person person;

	@Id
	@ManyToOne
	private Address address;

	//Getters and setters are omitted for brevity

	@Override
	public boolean equals(Object o) {
		if ( this == o ) {
			return true;
		}
		if ( o == null || getClass() != o.getClass() ) {
			return false;
		}
		PersonAddress that = (PersonAddress) o;
		return Objects.equals( person, that.person ) &&
				Objects.equals( address, that.address );
	}

	@Override
	public int hashCode() {
		return Objects.hash( person, address );
	}
}

@Entity(name = "Address")
public static class Address implements Serializable {

	@Id
	@GeneratedValue
	private Long id;

	private String street;

	@Column(name = "`number`")
	private String number;

	private String postalCode;

	@OneToMany(
		mappedBy = "address",
		cascade = CascadeType.ALL,
		orphanRemoval = true
	)
	private List<PersonAddress> owners = new ArrayList<>();

	//Getters and setters are omitted for brevity

	@Override
	public boolean equals(Object o) {
		if ( this == o ) {
			return true;
		}
		if ( o == null || getClass() != o.getClass() ) {
			return false;
		}
		Address address = (Address) o;
		return Objects.equals( street, address.street ) &&
				Objects.equals( number, address.number ) &&
				Objects.equals( postalCode, address.postalCode );
	}

	@Override
	public int hashCode() {
		return Objects.hash( street, number, postalCode );
	}
}
CREATE TABLE Address (
    id BIGINT NOT NULL ,
    number VARCHAR(255) ,
    postalCode VARCHAR(255) ,
    street VARCHAR(255) ,
    PRIMARY KEY ( id )
)

CREATE TABLE Person (
    id BIGINT NOT NULL ,
    registrationNumber VARCHAR(255) ,
    PRIMARY KEY ( id )
)

CREATE TABLE PersonAddress (
    person_id BIGINT NOT NULL ,
    address_id BIGINT NOT NULL ,
    PRIMARY KEY ( person_id, address_id )
)

ALTER TABLE Person
ADD CONSTRAINT UK_23enodonj49jm8uwec4i7y37f
UNIQUE (registrationNumber)

ALTER TABLE PersonAddress
ADD CONSTRAINT FK8b3lru5fyej1aarjflamwghqq
FOREIGN KEY (person_id) REFERENCES Person

ALTER TABLE PersonAddress
ADD CONSTRAINT FK7p69mgialumhegyl4byrh65jk
FOREIGN KEY (address_id) REFERENCES Address
```

