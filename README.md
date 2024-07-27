Got it! Let's revise the README to indicate that the project is under heavy development and that many features are coming soon. We'll also streamline it further to focus on the upcoming features and the overall vision.

---

<h1 align="center">Black Shield Redux ORM</h1>

<div align="center">

[![Build Status](https://img.shields.io/travis/redux-orm/redux-orm.svg?style=flat-square)](https://travis-ci.org/redux-orm/redux-orm)
[![Coverage Status](https://img.shields.io/codecov/c/github/redux-orm/redux-orm/master.svg?style=flat-square)](https://codecov.io/gh/redux-orm/redux-orm/branch/master)
[![NPM package](https://img.shields.io/npm/v/redux-orm.svg?style=flat-square)](https://www.npmjs.com/package/redux-orm)
![NPM package (next)](https://img.shields.io/npm/v/redux-orm/next?style=flat-square)
![GitHub Release Date](https://img.shields.io/github/release-date/redux-orm/redux-orm.svg?style=flat-square)
[![NPM downloads](https://img.shields.io/npm/dm/redux-orm.svg?style=flat-square)](https://www.npmjs.com/package/redux-orm)
[![Gitter](https://badges.gitter.im/redux-orm/Lobby.svg?style=flat-square)](https://gitter.im/redux-orm/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
![NPM license](https://img.shields.io/npm/l/redux-orm.svg?style=flat-square)
</div>

## Introduction

Black Shield Redux ORM is a modern, robust ORM library designed to manage relational data in your Redux store. This fork is a comprehensive rebuild of the original `redux-orm`, integrating advanced features, improved state change CRUD operations, and support for in-memory queries, IndexedDB, SQLite, Axios, Supabase, and Electron.js.

## Key Features (Coming Soon)

- **Advanced CRUD Operations**: Efficient and scalable state change handling.
- **In-Memory Queries**: Enhanced query performance for complex data relationships.
- **Database Support**: Seamless integration with IndexedDB, SQLite, and other storage solutions.
- **Integration with Axios and Supabase**: Simplified data fetching and synchronization.
- **Electron.js Compatibility**: Optimized for desktop applications.
- **Schema Models to Complex Query Methods**: Flexible and powerful data modeling.

## Installation

Currently, the project is under heavy development. Stay tuned for updates on installation and usage.

## Usage

### Declare Your Models

You can declare your models with the ES6 class syntax, extending from `Model`. Declare all your non-relational fields on the Model, and declaring all data fields is recommended to avoid redefining getters and setters when instantiating Models. Black Shield Redux ORM supports one-to-one and many-to-many relations in addition to foreign keys (`oneToOne`, `many`, and `fk` imports respectively).

```typescript
// models.ts
import { Model, fk, many, attr } from 'black-shield-redux-orm';

class Book extends Model {
    toString() {
        return `Book: ${this.name}`;
    }
}
Book.modelName = 'Book';

Book.fields = {
    id: attr(), // non-relational field for any value; optional but highly recommended
    name: attr(),
    publisherId: fk({ to: 'Publisher', as: 'publisher', relatedName: 'books' }),
    authors: many('Author', 'books'),
};

export default Book;
```

### Register Models and Generate an Empty Database State

Register all Models with an instance of the ORM class, which handles generating a full schema and passing that information to the database.

```typescript
// orm.ts
import { ORM } from 'black-shield-redux-orm';
import { Book, Author, Publisher } from './models';

const orm = new ORM({ stateSelector: state => state.orm });
orm.register(Book, Author, Publisher);

export default orm;
```

Generate an empty database state, which is a plain, nested JavaScript object structured similarly to relational databases.

```typescript
// index.ts
import orm from './orm';

const emptyDBState = orm.getEmptyState();
```

### Applying Updates to the Database

Start an ORM session on a database state to apply updates.

```typescript
const session = orm.session(emptyDBState);
const Book = session.Book;

Book.withId(1).update({ name: 'Clean Code' });
Book.all().filter(book => book.name === 'Clean Code').delete();
Book.idExists(1); // false

const updatedDBState = session.state;
```

## Redux Integration

To integrate Black Shield Redux ORM with Redux, define a reducer that instantiates a session from the database state held in the Redux state slice, apply your updates, and return the next state from the session.

```typescript
import orm from './orm';

function ormReducer(dbState, action) {
    const sess = orm.session(dbState);
    const { Book } = sess;

    switch (action.type) {
        case 'CREATE_BOOK':
            Book.create(action.payload);
            break;
        case 'UPDATE_BOOK':
            Book.withId(action.payload.id).update(action.payload);
            break;
        case 'REMOVE_BOOK':
            Book.withId(action.payload.id).delete();
            break;
        case 'ADD_AUTHOR_TO_BOOK':
            Book.withId(action.payload.bookId).authors.add(action.payload.author);
            break;
        case 'REMOVE_AUTHOR_FROM_BOOK':
            Book.withId(action.payload.bookId).authors.remove(action.payload.authorId);
            break;
        case 'ASSIGN_PUBLISHER':
            Book.withId(action.payload.bookId).publisherId = action.payload.publisherId;
            break;
    }

    return sess.state;
}
```

To define your update logic on the Model classes, specify a `reducer` static method on your model.

```typescript
class Book extends Model {
    static reducer(action, Book, session) {
        switch (action.type) {
            case 'CREATE_BOOK':
                Book.create(action.payload);
                break;
            case 'UPDATE_BOOK':
                Book.withId(action.payload.id).update(action.payload);
                break;
            case 'REMOVE_BOOK':
                const book = Book.withId(action.payload);
                book.delete();
                break;
            case 'ADD_AUTHOR_TO_BOOK':
                Book.withId(action.payload.bookId).authors.add(action.payload.author);
                break;
            case 'REMOVE_AUTHOR_FROM_BOOK':
                Book.withId(action.payload.bookId).authors.remove(action.payload.authorId);
                break;
            case 'ASSIGN_PUBLISHER':
                Book.withId(action.payload.bookId).publisherId = action.payload.publisherId;
                break;
        }
        return undefined;
    }

    toString() {
        return `Book: ${this.name}`;
    }
}
```

## Credits and License

Black Shield Redux ORM builds upon the foundation laid by the original `redux-orm` project. We extend our gratitude to the contributors of `redux-orm` for their work.

MIT. See [`LICENSE`](https://github.com/redux-orm/redux-orm/blob/master/LICENSE).

---

This README indicates that the project is still in development while providing an overview of its future capabilities.
