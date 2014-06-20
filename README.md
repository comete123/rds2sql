# RDS2SQL (Generate SQL code from a UML and/or RDS model)

RDS2SQL is a UML/RDS to SQL generator written in Acceleo and QVTo. This generator provides a 
way to generate your Database schema from a UML class diagram. It focuses on table-field-primary key-foreign key
creation.

## Translation 

Here is the mapping which is applied to UML in order to produce the SQL code.

| UML | SQL |
| :-- | :-- |
|Class (abstract or not) | Table |
|Attribute | Field |
|Association | Relationship in the SQL sense (*i.e.*, a field and a foreign key + table in *many-to-many* cases) |
|Multiplicity/cardinality | constraint on field and on foreign key location |
|UML primitive types | SQL types (the ones that can be translated) |

## Preconditions/Translation Choices

Some choices were made to differenciate elements during translation:

- The first attribute in the class is considered as the primary key.
- A Class has to own at least one attribute (*i.e*, one primary key) if it is involved in a relationship with
another Class.
- Managed attribute types are the primitive ones (UnlimitedNatural, Boolean, ByteArray, Char, Date, Long, Real, String, Integer)
- If an attribute has an undefined type, it is not generated.
- If an attribute has no name, it is not generated.
- Associations are managed according to their association end multiplicity.
- Abstract classes are generated as tables.
 
## Known Limitations

- Inheritance is not currently managed.
- Interface and Enumeration are not translated.
- Comment are not currently supported.

## GenMyModel Integration

You can directly integrate this generator into [GenMyModel](http://www.genmymodel.com "GenMyModel Website") using
custom generators. 

Clic on `Tools` then `Custom generator`. Add a custom gen and enter the following keys:

       Generator name: RDS2SQL (or whatever you want actually)
       Github URL:     https://github.com/gourch/rds2sql.git
       Github branch:  master (not required)

As this generator is public on Github, there is no need to enter login information. Do not forget to save it and
that's all. You are good to go! You can launch the generator and download the code source (`zip` format).

## Options

You can parameter the code generation by adding options. Currently, `4` dedicated options are managed by the
generator:

- `MYSQL` to generate MySQL like code
- `POSTGRESQL` to generate Postgresql like code
- `H2DB` to generate H2DB like code
- `ORACLE` to generate Oracle like code

If no option is chosen, a more general SQL code is produced (seems to work for MSSQL and some other SQL dialect, but not tested in depth).

## Uml2SQL Technical Overview

This generator follows three main phases. The first two are
written in QVTo. They refine the input UML model in order to bring
it to a more SQLish representation.

Once the UML model is refined, code is generated by an Acceleo (MTL)
transformation.

## Details About Generation Phases

The three generation phases have distinct purposes. In order to help
you jump in, here is a small explanation of what they are used for.

### From UML to SQL

As you probably know, directly generating code from UML could be cumbersome (a nice and polite way of
saying it could be a real pain in the ass). So, a first QVTo transformation named `uml2sql.qvto` is used to go from UML
models, that conforms to the UML Metamodel, to SQL models that conforms to the SQL Metamodel, defined in `metamodels` 
folder in source code, named `genericSql.ecore` (Ecore EMF Format). 

A quick word about the SQL Metamodel. It was defined as a physical Database view and not as a logical view. Moreover,
as it is used to generate code from UML class diagrams, it only deals with `TABLE` creation (not `VIEW` and other stuff
like that).

### From SQL to SQLish SQL

Okay, this one is a tricky one. In order to deal with different SQL dialects (Oracle, MySQL...etc), __for our purpose__, the
only thing which changes between dialect seems to be the type. So, a new QVTo transformation (dedicated to a 
specific dialect) is launched. Each transformation refines the generic SQL types that are present in the model to a specific
one. This specific type is the one used during code generation.

### From SQL Model to SQL Code

This is the last step. The `rds2sql.mtl` code generation translates the SQL Model into code. This code generation is written
in `52` lines. The number of lines had been drastically reduced by the previous QVTo transformations. At this last step, as
the model is already defined, we just have to translate it into code. No more intelligence is required.

Here is an example of generated code (from this UML model <http://repository.genmymodel.com/ali.gourch/Example>)

```sql
CREATE TABLE Company
(
        nSiret VARCHAR(100) NOT NULL UNIQUE,
        name VARCHAR(100) NOT NULL UNIQUE,
        creation DATE NOT NULL ,
        PRIMARY KEY(nSiret)
);

CREATE TABLE Person
(
        id INT NOT NULL UNIQUE,
        name VARCHAR(100) NOT NULL ,
        adress VARCHAR(100) NOT NULL ,
        gender BOOLEAN NOT NULL ,
        PRIMARY KEY(id)
);

CREATE TABLE Company_Person
(
        company_nSiret VARCHAR(100),
        person_id INT,
        PRIMARY KEY(company_nSiret, person_id)
);

ALTER TABLE Company_Person
        ADD CONSTRAINT company
                FOREIGN KEY(company_nSiret)
                        REFERENCES Company(nSiret); 
ALTER TABLE Company_Person
        ADD CONSTRAINT person
                FOREIGN KEY(person_id)
                        REFERENCES Person(id);
```
     
     
