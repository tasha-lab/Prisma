# Prisma

An ORM (Object-Relational Mapping) reduces working with databases to offering a type safe query API and support for relational databases.

### Installing Prisma

To insatll Prisma:

- First you need to have a package.json file. To install, you run <code> npm init -y</code>
- Then you need to install prisma as a develpment dependency using <code>npm install prisma -D</code>

### Inititalizing prisma

To set up prisma run <code>npx prisma init --datasource-provider Database</code> to create a prisma directory and a .env file.  
The database part is replaced by the database you are using.Prisma uses postgresql by default, meaning that when you run <code>npx prisma init</code>, its going to work the same way.

### Set up a database

The .env file comes with a database connection. You need to configure details like your username,password, host, port and database name.  
It should look something like, <code>DATABASE_URL="postgresql://postgres:1234@localhost:5432/products"</code>

## Models

Models define database table with fields, datatypes and constraints.

### Models consists of

- **Data models**- Columns in the database eg. age, f_name
- **Field Types**- Datatypes of each column eg. string, boolean
- **Relations**- How models relate to each other using relation fields
- **Attributese**- Add additional data using data eg. @id, @default

### Fields consists of:

- Field name
- Field's Data type
- Field type modifier. The two important modifiers are <code>?</code> and <code>[]</code>.

  - <code>?</code> - Specify whether a field is optional
  - <code>[]</code>- Whether a field should contain multiple values

- Attributes

### Creating models

To create a model, you run a code similar to these. The attributes are optional but field names must be there.

```model User {
    id String @id @default(uuid())
    name String?
    email String @unique
    role Role @default(USER)
}
```

### Migration

Migrations apply schema changes to you database. To run a migration, run <code>npx prisma migrate dev --name migration name</code>

## Prisma client

A prisma client is generated to be used in production to interact with postgresql server

### Install Prisma client

It is used to interact with the database. If you did everything right it should be available, if not you can generate and install it manually.  
Run <code>npx prisma generate</code> to generate and <code>npm install @prisma/client</code> to install

## View the database

To view the database you can view the database you specified earlier in the SQL shell(psql) and run \l to see the list of databases you have then, <code>\d database_name</code> to view your table.  
Alternatively, for better visualization you can use Prisma studio. For this run <code>npx prisma studio</code> and access it from <i>localhost:5555</i>

## CRUD Operations

These CRUD operations are; **Create**,**Read**,**Update** and **Delete** using prisma client.

### **Create**

It is used to insert new data. The data can be passed in an object. You can use <code>create()</code> which returns the object and the number of objects that have been created or <code>createMany()</code> which only returns the objects.

```
const newUser = await prisma.user.create({
    data: {
        name: 'Pam',
        email: 'pam@paper.com',
        age: 26,
    },
});
```

### **Read**

To get all records, use <code>findMany()</code>

```
const filteredUsers = await prisma.user.findMany({
    where: {
        name: { contains: 'a' },
        age: { gt: 30 },
    },
});
```

To find the first record that matches a criteria,<code>findFirst()</code> with the filter <code>where</code>  
You can specify the field you want returned use <code>select</code>  
You can use AND operator to select records matching multiple criterias  
You can use the OR operator to select records matching any criteria

### **Update**

You can update a single record using <code>update()</code>  
You can also update many records using <code>updateMany()</code>

```
const updatedUser = await prisma.user.update({
    where: { email: 'pam@paper.com' },
    data: { age: { increment: 1 } },
});
```

### **Delete**

To delete a single record use <code>delete()</code>  
For multiple records use <code>deleteMany()</code>

```
const deletedUser = await prisma.user.delete({
    where: { email: 'pam@paper.com' },
});
```

```
const deletedUserCount = await prisma.user.deleteMany({
  where: {
    email: {
      contains: 'prisma.io',
    },
  },
  limit: 5,
});
```

## Relationships

To define relationships between models use <code>@relation</code>.  
There are multiple relations supported by prisma ie.

- One-to-one(1-1)- A record in a table is associated to only one record in another table

```
model User {
  id      Int      @id @default(autoincrement())
  profile Profile?
}

model Profile {
  id     Int  @id @default(autoincrement())
  user   User @relation(fields: [userId], references: [id])
  userId Int  @unique // relation scalar field (used in the `@relation` attribute above)
}
```

- One-to-many(1-n)- A record in one table can be associated with multiple other records in another table

```
model Post {
  id         Int                 @id @default(autoincrement())
  title      String
  categories CategoriesOnPosts[]
}

model Category {
  id    Int                 @id @default(autoincrement())
  name  String
  posts CategoriesOnPosts[]
}

model CategoriesOnPosts {
  post       Post     @relation(fields: [postId], references: [id])
  postId     Int // relation scalar field (used in the `@relation` attribute above)
  category   Category @relation(fields: [categoryId], references: [id])
  categoryId Int // relation scalar field (used in the `@relation` attribute above)
  assignedAt DateTime @default(now())
  assignedBy String

  @@id([postId, categoryId])
}
```

- Many-to-many(m-n)- Multiple records in one table can be associated to multiple records in another table
