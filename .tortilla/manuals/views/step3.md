# Step 3: Mutations

[//]: # (head-end)


# Chapter 5

First, let's start with the server.
We'll need to install a couple of packages:

    $ npm install @koa/cors apollo-server-koa graphql graphql-tools koa koa-bodyparser koa-router
    $ npm install --save-dev @types/graphql @types/koa @types/koa-bodyparser @types/koa-router @types/koa__cors

Now let's create some empty schemas and resolvers:

[{]: <helper> (diffStep "1.1" files="schema/*")

#### [Step 1.1: Create empty Apollo server](https://github.com/Urigo/WhatsApp-Clone-Server/commit/0e91909)

##### Added schema&#x2F;index.ts
```diff
@@ -0,0 +1,8 @@
+┊ ┊1┊import { makeExecutableSchema } from 'apollo-server-express';
+┊ ┊2┊import typeDefs from './typeDefs';
+┊ ┊3┊import { resolvers } from './resolvers';
+┊ ┊4┊
+┊ ┊5┊export const schema = makeExecutableSchema({
+┊ ┊6┊  typeDefs,
+┊ ┊7┊  resolvers,
+┊ ┊8┊});
```

##### Added schema&#x2F;resolvers.ts
```diff
@@ -0,0 +1,5 @@
+┊ ┊1┊import { IResolvers } from 'apollo-server-express';
+┊ ┊2┊
+┊ ┊3┊export const resolvers: IResolvers = {
+┊ ┊4┊  Query: {},
+┊ ┊5┊};
```

##### Added schema&#x2F;typeDefs.ts
```diff
@@ -0,0 +1,2 @@
+┊ ┊1┊export default `
+┊ ┊2┊`;
```

[}]: #

Time to create our index:

[{]: <helper> (diffStep "1.1" files="^index.ts")

#### [Step 1.1: Create empty Apollo server](https://github.com/Urigo/WhatsApp-Clone-Server/commit/0e91909)

##### Added index.ts
```diff
@@ -0,0 +1,23 @@
+┊  ┊ 1┊import { schema } from "./schema";
+┊  ┊ 2┊import bodyParser from "body-parser";
+┊  ┊ 3┊import cors from 'cors';
+┊  ┊ 4┊import express from 'express';
+┊  ┊ 5┊import { ApolloServer } from "apollo-server-express";
+┊  ┊ 6┊
+┊  ┊ 7┊const PORT = 4000;
+┊  ┊ 8┊
+┊  ┊ 9┊const app = express();
+┊  ┊10┊
+┊  ┊11┊app.use(cors());
+┊  ┊12┊app.use(bodyParser.json());
+┊  ┊13┊
+┊  ┊14┊const apollo = new ApolloServer({
+┊  ┊15┊  schema
+┊  ┊16┊});
+┊  ┊17┊
+┊  ┊18┊apollo.applyMiddleware({
+┊  ┊19┊  app,
+┊  ┊20┊  path: '/graphql'
+┊  ┊21┊});
+┊  ┊22┊
+┊  ┊23┊app.listen(PORT);
```

[}]: #

Now we want to feed our graphql server with some data, so let's install moment

    $ npm install moment

and create a fake db:

[{]: <helper> (diffStep "1.2" files="db.ts")

#### [Step 1.2: Add fake db](https://github.com/Urigo/WhatsApp-Clone-Server/commit/0202540)

##### Added db.ts
```diff
@@ -0,0 +1,448 @@
+┊   ┊  1┊import moment from 'moment';
+┊   ┊  2┊
+┊   ┊  3┊export enum MessageType {
+┊   ┊  4┊  PICTURE,
+┊   ┊  5┊  TEXT,
+┊   ┊  6┊  LOCATION,
+┊   ┊  7┊}
+┊   ┊  8┊
+┊   ┊  9┊export interface UserDb {
+┊   ┊ 10┊  id: number,
+┊   ┊ 11┊  username: string,
+┊   ┊ 12┊  password: string,
+┊   ┊ 13┊  name: string,
+┊   ┊ 14┊  picture?: string | null,
+┊   ┊ 15┊  phone?: string | null,
+┊   ┊ 16┊}
+┊   ┊ 17┊
+┊   ┊ 18┊export interface ChatDb {
+┊   ┊ 19┊  id: number,
+┊   ┊ 20┊  createdAt: Date,
+┊   ┊ 21┊  name?: string | null,
+┊   ┊ 22┊  picture?: string | null,
+┊   ┊ 23┊  // All members, current and past ones.
+┊   ┊ 24┊  allTimeMemberIds: number[],
+┊   ┊ 25┊  // Whoever gets the chat listed. For groups includes past members who still didn't delete the group.
+┊   ┊ 26┊  listingMemberIds: number[],
+┊   ┊ 27┊  // Actual members of the group (they are not the only ones who get the group listed). Null for chats.
+┊   ┊ 28┊  actualGroupMemberIds?: number[] | null,
+┊   ┊ 29┊  adminIds?: number[] | null,
+┊   ┊ 30┊  ownerId?: number | null,
+┊   ┊ 31┊  messages: MessageDb[],
+┊   ┊ 32┊}
+┊   ┊ 33┊
+┊   ┊ 34┊export interface MessageDb {
+┊   ┊ 35┊  id: number,
+┊   ┊ 36┊  chatId: number,
+┊   ┊ 37┊  senderId: number,
+┊   ┊ 38┊  content: string,
+┊   ┊ 39┊  createdAt: Date,
+┊   ┊ 40┊  type: MessageType,
+┊   ┊ 41┊  recipients: RecipientDb[],
+┊   ┊ 42┊  holderIds: number[],
+┊   ┊ 43┊}
+┊   ┊ 44┊
+┊   ┊ 45┊export interface RecipientDb {
+┊   ┊ 46┊  userId: number,
+┊   ┊ 47┊  messageId: number,
+┊   ┊ 48┊  chatId: number,
+┊   ┊ 49┊  receivedAt: Date | null,
+┊   ┊ 50┊  readAt: Date | null,
+┊   ┊ 51┊}
+┊   ┊ 52┊
+┊   ┊ 53┊const users: UserDb[] = [
+┊   ┊ 54┊  {
+┊   ┊ 55┊    id: 1,
+┊   ┊ 56┊    username: 'ethan',
+┊   ┊ 57┊    password: '$2a$08$NO9tkFLCoSqX1c5wk3s7z.JfxaVMKA.m7zUDdDwEquo4rvzimQeJm', // 111
+┊   ┊ 58┊    name: 'Ethan Gonzalez',
+┊   ┊ 59┊    picture: 'https://randomuser.me/api/portraits/thumb/men/1.jpg',
+┊   ┊ 60┊    phone: '+391234567890',
+┊   ┊ 61┊  },
+┊   ┊ 62┊  {
+┊   ┊ 63┊    id: 2,
+┊   ┊ 64┊    username: 'bryan',
+┊   ┊ 65┊    password: '$2a$08$xE4FuCi/ifxjL2S8CzKAmuKLwv18ktksSN.F3XYEnpmcKtpbpeZgO', // 222
+┊   ┊ 66┊    name: 'Bryan Wallace',
+┊   ┊ 67┊    picture: 'https://randomuser.me/api/portraits/thumb/men/2.jpg',
+┊   ┊ 68┊    phone: '+391234567891',
+┊   ┊ 69┊  },
+┊   ┊ 70┊  {
+┊   ┊ 71┊    id: 3,
+┊   ┊ 72┊    username: 'avery',
+┊   ┊ 73┊    password: '$2a$08$UHgH7J8G6z1mGQn2qx2kdeWv0jvgHItyAsL9hpEUI3KJmhVW5Q1d.', // 333
+┊   ┊ 74┊    name: 'Avery Stewart',
+┊   ┊ 75┊    picture: 'https://randomuser.me/api/portraits/thumb/women/1.jpg',
+┊   ┊ 76┊    phone: '+391234567892',
+┊   ┊ 77┊  },
+┊   ┊ 78┊  {
+┊   ┊ 79┊    id: 4,
+┊   ┊ 80┊    username: 'katie',
+┊   ┊ 81┊    password: '$2a$08$wR1k5Q3T9FC7fUgB7Gdb9Os/GV7dGBBf4PLlWT7HERMFhmFDt47xi', // 444
+┊   ┊ 82┊    name: 'Katie Peterson',
+┊   ┊ 83┊    picture: 'https://randomuser.me/api/portraits/thumb/women/2.jpg',
+┊   ┊ 84┊    phone: '+391234567893',
+┊   ┊ 85┊  },
+┊   ┊ 86┊  {
+┊   ┊ 87┊    id: 5,
+┊   ┊ 88┊    username: 'ray',
+┊   ┊ 89┊    password: '$2a$08$6.mbXqsDX82ZZ7q5d8Osb..JrGSsNp4R3IKj7mxgF6YGT0OmMw242', // 555
+┊   ┊ 90┊    name: 'Ray Edwards',
+┊   ┊ 91┊    picture: 'https://randomuser.me/api/portraits/thumb/men/3.jpg',
+┊   ┊ 92┊    phone: '+391234567894',
+┊   ┊ 93┊  },
+┊   ┊ 94┊  {
+┊   ┊ 95┊    id: 6,
+┊   ┊ 96┊    username: 'niko',
+┊   ┊ 97┊    password: '$2a$08$fL5lZR.Rwf9FWWe8XwwlceiPBBim8n9aFtaem.INQhiKT4.Ux3Uq.', // 666
+┊   ┊ 98┊    name: 'Niccolò Belli',
+┊   ┊ 99┊    picture: 'https://randomuser.me/api/portraits/thumb/men/4.jpg',
+┊   ┊100┊    phone: '+391234567895',
+┊   ┊101┊  },
+┊   ┊102┊  {
+┊   ┊103┊    id: 7,
+┊   ┊104┊    username: 'mario',
+┊   ┊105┊    password: '$2a$08$nDHDmWcVxDnH5DDT3HMMC.psqcnu6wBiOgkmJUy9IH..qxa3R6YrO', // 777
+┊   ┊106┊    name: 'Mario Rossi',
+┊   ┊107┊    picture: 'https://randomuser.me/api/portraits/thumb/men/5.jpg',
+┊   ┊108┊    phone: '+391234567896',
+┊   ┊109┊  },
+┊   ┊110┊];
+┊   ┊111┊
+┊   ┊112┊const chats: ChatDb[] = [
+┊   ┊113┊  {
+┊   ┊114┊    id: 1,
+┊   ┊115┊    createdAt: moment().subtract(10, 'weeks').toDate(),
+┊   ┊116┊    name: null,
+┊   ┊117┊    picture: null,
+┊   ┊118┊    allTimeMemberIds: [1, 3],
+┊   ┊119┊    listingMemberIds: [1, 3],
+┊   ┊120┊    adminIds: null,
+┊   ┊121┊    ownerId: null,
+┊   ┊122┊    messages: [
+┊   ┊123┊      {
+┊   ┊124┊        id: 1,
+┊   ┊125┊        chatId: 1,
+┊   ┊126┊        senderId: 1,
+┊   ┊127┊        content: 'You on your way?',
+┊   ┊128┊        createdAt: moment().subtract(1, 'hours').toDate(),
+┊   ┊129┊        type: MessageType.TEXT,
+┊   ┊130┊        recipients: [
+┊   ┊131┊          {
+┊   ┊132┊            userId: 3,
+┊   ┊133┊            messageId: 1,
+┊   ┊134┊            chatId: 1,
+┊   ┊135┊            receivedAt: null,
+┊   ┊136┊            readAt: null,
+┊   ┊137┊          },
+┊   ┊138┊        ],
+┊   ┊139┊        holderIds: [1, 3],
+┊   ┊140┊      },
+┊   ┊141┊      {
+┊   ┊142┊        id: 2,
+┊   ┊143┊        chatId: 1,
+┊   ┊144┊        senderId: 3,
+┊   ┊145┊        content: 'Yep!',
+┊   ┊146┊        createdAt: moment().subtract(1, 'hours').add(5, 'minutes').toDate(),
+┊   ┊147┊        type: MessageType.TEXT,
+┊   ┊148┊        recipients: [
+┊   ┊149┊          {
+┊   ┊150┊            userId: 1,
+┊   ┊151┊            messageId: 2,
+┊   ┊152┊            chatId: 1,
+┊   ┊153┊            receivedAt: null,
+┊   ┊154┊            readAt: null,
+┊   ┊155┊          },
+┊   ┊156┊        ],
+┊   ┊157┊        holderIds: [3, 1],
+┊   ┊158┊      },
+┊   ┊159┊    ],
+┊   ┊160┊  },
+┊   ┊161┊  {
+┊   ┊162┊    id: 2,
+┊   ┊163┊    createdAt: moment().subtract(9, 'weeks').toDate(),
+┊   ┊164┊    name: null,
+┊   ┊165┊    picture: null,
+┊   ┊166┊    allTimeMemberIds: [1, 4],
+┊   ┊167┊    listingMemberIds: [1, 4],
+┊   ┊168┊    adminIds: null,
+┊   ┊169┊    ownerId: null,
+┊   ┊170┊    messages: [
+┊   ┊171┊      {
+┊   ┊172┊        id: 1,
+┊   ┊173┊        chatId: 2,
+┊   ┊174┊        senderId: 1,
+┊   ┊175┊        content: 'Hey, it\'s me',
+┊   ┊176┊        createdAt: moment().subtract(2, 'hours').toDate(),
+┊   ┊177┊        type: MessageType.TEXT,
+┊   ┊178┊        recipients: [
+┊   ┊179┊          {
+┊   ┊180┊            userId: 4,
+┊   ┊181┊            messageId: 1,
+┊   ┊182┊            chatId: 2,
+┊   ┊183┊            receivedAt: null,
+┊   ┊184┊            readAt: null,
+┊   ┊185┊          },
+┊   ┊186┊        ],
+┊   ┊187┊        holderIds: [1, 4],
+┊   ┊188┊      },
+┊   ┊189┊    ],
+┊   ┊190┊  },
+┊   ┊191┊  {
+┊   ┊192┊    id: 3,
+┊   ┊193┊    createdAt: moment().subtract(8, 'weeks').toDate(),
+┊   ┊194┊    name: null,
+┊   ┊195┊    picture: null,
+┊   ┊196┊    allTimeMemberIds: [1, 5],
+┊   ┊197┊    listingMemberIds: [1, 5],
+┊   ┊198┊    adminIds: null,
+┊   ┊199┊    ownerId: null,
+┊   ┊200┊    messages: [
+┊   ┊201┊      {
+┊   ┊202┊        id: 1,
+┊   ┊203┊        chatId: 3,
+┊   ┊204┊        senderId: 1,
+┊   ┊205┊        content: 'I should buy a boat',
+┊   ┊206┊        createdAt: moment().subtract(1, 'days').toDate(),
+┊   ┊207┊        type: MessageType.TEXT,
+┊   ┊208┊        recipients: [
+┊   ┊209┊          {
+┊   ┊210┊            userId: 5,
+┊   ┊211┊            messageId: 1,
+┊   ┊212┊            chatId: 3,
+┊   ┊213┊            receivedAt: null,
+┊   ┊214┊            readAt: null,
+┊   ┊215┊          },
+┊   ┊216┊        ],
+┊   ┊217┊        holderIds: [1, 5],
+┊   ┊218┊      },
+┊   ┊219┊      {
+┊   ┊220┊        id: 2,
+┊   ┊221┊        chatId: 3,
+┊   ┊222┊        senderId: 1,
+┊   ┊223┊        content: 'You still there?',
+┊   ┊224┊        createdAt: moment().subtract(1, 'days').add(16, 'hours').toDate(),
+┊   ┊225┊        type: MessageType.TEXT,
+┊   ┊226┊        recipients: [
+┊   ┊227┊          {
+┊   ┊228┊            userId: 5,
+┊   ┊229┊            messageId: 2,
+┊   ┊230┊            chatId: 3,
+┊   ┊231┊            receivedAt: null,
+┊   ┊232┊            readAt: null,
+┊   ┊233┊          },
+┊   ┊234┊        ],
+┊   ┊235┊        holderIds: [1, 5],
+┊   ┊236┊      },
+┊   ┊237┊    ],
+┊   ┊238┊  },
+┊   ┊239┊  {
+┊   ┊240┊    id: 4,
+┊   ┊241┊    createdAt: moment().subtract(7, 'weeks').toDate(),
+┊   ┊242┊    name: null,
+┊   ┊243┊    picture: null,
+┊   ┊244┊    allTimeMemberIds: [3, 4],
+┊   ┊245┊    listingMemberIds: [3, 4],
+┊   ┊246┊    adminIds: null,
+┊   ┊247┊    ownerId: null,
+┊   ┊248┊    messages: [
+┊   ┊249┊      {
+┊   ┊250┊        id: 1,
+┊   ┊251┊        chatId: 4,
+┊   ┊252┊        senderId: 3,
+┊   ┊253┊        content: 'Look at my mukluks!',
+┊   ┊254┊        createdAt: moment().subtract(4, 'days').toDate(),
+┊   ┊255┊        type: MessageType.TEXT,
+┊   ┊256┊        recipients: [
+┊   ┊257┊          {
+┊   ┊258┊            userId: 4,
+┊   ┊259┊            messageId: 1,
+┊   ┊260┊            chatId: 4,
+┊   ┊261┊            receivedAt: null,
+┊   ┊262┊            readAt: null,
+┊   ┊263┊          },
+┊   ┊264┊        ],
+┊   ┊265┊        holderIds: [3, 4],
+┊   ┊266┊      },
+┊   ┊267┊    ],
+┊   ┊268┊  },
+┊   ┊269┊  {
+┊   ┊270┊    id: 5,
+┊   ┊271┊    createdAt: moment().subtract(6, 'weeks').toDate(),
+┊   ┊272┊    name: null,
+┊   ┊273┊    picture: null,
+┊   ┊274┊    allTimeMemberIds: [2, 5],
+┊   ┊275┊    listingMemberIds: [2, 5],
+┊   ┊276┊    adminIds: null,
+┊   ┊277┊    ownerId: null,
+┊   ┊278┊    messages: [
+┊   ┊279┊      {
+┊   ┊280┊        id: 1,
+┊   ┊281┊        chatId: 5,
+┊   ┊282┊        senderId: 2,
+┊   ┊283┊        content: 'This is wicked good ice cream.',
+┊   ┊284┊        createdAt: moment().subtract(2, 'weeks').toDate(),
+┊   ┊285┊        type: MessageType.TEXT,
+┊   ┊286┊        recipients: [
+┊   ┊287┊          {
+┊   ┊288┊            userId: 5,
+┊   ┊289┊            messageId: 1,
+┊   ┊290┊            chatId: 5,
+┊   ┊291┊            receivedAt: null,
+┊   ┊292┊            readAt: null,
+┊   ┊293┊          },
+┊   ┊294┊        ],
+┊   ┊295┊        holderIds: [2, 5],
+┊   ┊296┊      },
+┊   ┊297┊      {
+┊   ┊298┊        id: 2,
+┊   ┊299┊        chatId: 6,
+┊   ┊300┊        senderId: 5,
+┊   ┊301┊        content: 'Love it!',
+┊   ┊302┊        createdAt: moment().subtract(2, 'weeks').add(10, 'minutes').toDate(),
+┊   ┊303┊        type: MessageType.TEXT,
+┊   ┊304┊        recipients: [
+┊   ┊305┊          {
+┊   ┊306┊            userId: 2,
+┊   ┊307┊            messageId: 2,
+┊   ┊308┊            chatId: 5,
+┊   ┊309┊            receivedAt: null,
+┊   ┊310┊            readAt: null,
+┊   ┊311┊          },
+┊   ┊312┊        ],
+┊   ┊313┊        holderIds: [5, 2],
+┊   ┊314┊      },
+┊   ┊315┊    ],
+┊   ┊316┊  },
+┊   ┊317┊  {
+┊   ┊318┊    id: 6,
+┊   ┊319┊    createdAt: moment().subtract(5, 'weeks').toDate(),
+┊   ┊320┊    name: null,
+┊   ┊321┊    picture: null,
+┊   ┊322┊    allTimeMemberIds: [1, 6],
+┊   ┊323┊    listingMemberIds: [1],
+┊   ┊324┊    adminIds: null,
+┊   ┊325┊    ownerId: null,
+┊   ┊326┊    messages: [],
+┊   ┊327┊  },
+┊   ┊328┊  {
+┊   ┊329┊    id: 7,
+┊   ┊330┊    createdAt: moment().subtract(4, 'weeks').toDate(),
+┊   ┊331┊    name: null,
+┊   ┊332┊    picture: null,
+┊   ┊333┊    allTimeMemberIds: [2, 1],
+┊   ┊334┊    listingMemberIds: [2],
+┊   ┊335┊    adminIds: null,
+┊   ┊336┊    ownerId: null,
+┊   ┊337┊    messages: [],
+┊   ┊338┊  },
+┊   ┊339┊  {
+┊   ┊340┊    id: 8,
+┊   ┊341┊    createdAt: moment().subtract(3, 'weeks').toDate(),
+┊   ┊342┊    name: 'A user 0 group',
+┊   ┊343┊    picture: 'https://randomuser.me/api/portraits/thumb/lego/1.jpg',
+┊   ┊344┊    allTimeMemberIds: [1, 3, 4, 6],
+┊   ┊345┊    listingMemberIds: [1, 3, 4, 6],
+┊   ┊346┊    actualGroupMemberIds: [1, 4, 6],
+┊   ┊347┊    adminIds: [1, 6],
+┊   ┊348┊    ownerId: 1,
+┊   ┊349┊    messages: [
+┊   ┊350┊      {
+┊   ┊351┊        id: 1,
+┊   ┊352┊        chatId: 8,
+┊   ┊353┊        senderId: 1,
+┊   ┊354┊        content: 'I made a group',
+┊   ┊355┊        createdAt: moment().subtract(2, 'weeks').toDate(),
+┊   ┊356┊        type: MessageType.TEXT,
+┊   ┊357┊        recipients: [
+┊   ┊358┊          {
+┊   ┊359┊            userId: 3,
+┊   ┊360┊            messageId: 1,
+┊   ┊361┊            chatId: 8,
+┊   ┊362┊            receivedAt: null,
+┊   ┊363┊            readAt: null,
+┊   ┊364┊          },
+┊   ┊365┊          {
+┊   ┊366┊            userId: 4,
+┊   ┊367┊            messageId: 1,
+┊   ┊368┊            chatId: 8,
+┊   ┊369┊            receivedAt: moment().subtract(2, 'weeks').add(1, 'minutes').toDate(),
+┊   ┊370┊            readAt: moment().subtract(2, 'weeks').add(5, 'minutes').toDate(),
+┊   ┊371┊          },
+┊   ┊372┊          {
+┊   ┊373┊            userId: 6,
+┊   ┊374┊            messageId: 1,
+┊   ┊375┊            chatId: 8,
+┊   ┊376┊            receivedAt: null,
+┊   ┊377┊            readAt: null,
+┊   ┊378┊          },
+┊   ┊379┊        ],
+┊   ┊380┊        holderIds: [1, 3, 4, 6],
+┊   ┊381┊      },
+┊   ┊382┊      {
+┊   ┊383┊        id: 2,
+┊   ┊384┊        chatId: 8,
+┊   ┊385┊        senderId: 1,
+┊   ┊386┊        content: 'Ops, user 3 was not supposed to be here',
+┊   ┊387┊        createdAt: moment().subtract(2, 'weeks').add(2, 'minutes').toDate(),
+┊   ┊388┊        type: MessageType.TEXT,
+┊   ┊389┊        recipients: [
+┊   ┊390┊          {
+┊   ┊391┊            userId: 4,
+┊   ┊392┊            messageId: 2,
+┊   ┊393┊            chatId: 8,
+┊   ┊394┊            receivedAt: moment().subtract(2, 'weeks').add(3, 'minutes').toDate(),
+┊   ┊395┊            readAt: moment().subtract(2, 'weeks').add(5, 'minutes').toDate(),
+┊   ┊396┊          },
+┊   ┊397┊          {
+┊   ┊398┊            userId: 6,
+┊   ┊399┊            messageId: 2,
+┊   ┊400┊            chatId: 8,
+┊   ┊401┊            receivedAt: null,
+┊   ┊402┊            readAt: null,
+┊   ┊403┊          },
+┊   ┊404┊        ],
+┊   ┊405┊        holderIds: [1, 4, 6],
+┊   ┊406┊      },
+┊   ┊407┊      {
+┊   ┊408┊        id: 3,
+┊   ┊409┊        chatId: 8,
+┊   ┊410┊        senderId: 4,
+┊   ┊411┊        content: 'Awesome!',
+┊   ┊412┊        createdAt: moment().subtract(2, 'weeks').add(10, 'minutes').toDate(),
+┊   ┊413┊        type: MessageType.TEXT,
+┊   ┊414┊        recipients: [
+┊   ┊415┊          {
+┊   ┊416┊            userId: 1,
+┊   ┊417┊            messageId: 3,
+┊   ┊418┊            chatId: 8,
+┊   ┊419┊            receivedAt: null,
+┊   ┊420┊            readAt: null,
+┊   ┊421┊          },
+┊   ┊422┊          {
+┊   ┊423┊            userId: 6,
+┊   ┊424┊            messageId: 3,
+┊   ┊425┊            chatId: 8,
+┊   ┊426┊            receivedAt: null,
+┊   ┊427┊            readAt: null,
+┊   ┊428┊          },
+┊   ┊429┊        ],
+┊   ┊430┊        holderIds: [1, 4, 6],
+┊   ┊431┊      },
+┊   ┊432┊    ],
+┊   ┊433┊  },
+┊   ┊434┊  {
+┊   ┊435┊    id: 9,
+┊   ┊436┊    createdAt: moment().subtract(2, 'weeks').toDate(),
+┊   ┊437┊    name: 'A user 5 group',
+┊   ┊438┊    picture: null,
+┊   ┊439┊    allTimeMemberIds: [6, 3],
+┊   ┊440┊    listingMemberIds: [6, 3],
+┊   ┊441┊    actualGroupMemberIds: [6, 3],
+┊   ┊442┊    adminIds: [6],
+┊   ┊443┊    ownerId: 6,
+┊   ┊444┊    messages: [],
+┊   ┊445┊  },
+┊   ┊446┊];
+┊   ┊447┊
+┊   ┊448┊export const db = {users, chats};
```

[}]: #

Its' time to create our schema and our resolvers:

[{]: <helper> (diffStep "1.3")

#### [Step 1.3: Add resolvers and schema](https://github.com/Urigo/WhatsApp-Clone-Server/commit/861d01d)

##### Changed package.json
```diff
@@ -16,6 +16,7 @@
 ┊16┊16┊    "@types/cors": "^2.8.4",
 ┊17┊17┊    "@types/express": "^4.16.1",
 ┊18┊18┊    "@types/graphql": "^14.0.7",
+┊  ┊19┊    "@types/graphql-iso-date": "^3.3.1",
 ┊19┊20┊    "@types/node": "^11.9.5",
 ┊20┊21┊    "nodemon": "^1.18.10",
 ┊21┊22┊    "ts-node": "^8.0.2",
```
```diff
@@ -27,6 +28,7 @@
 ┊27┊28┊    "cors": "^2.8.5",
 ┊28┊29┊    "express": "^4.16.4",
 ┊29┊30┊    "graphql": "^14.1.1",
+┊  ┊31┊    "graphql-iso-date": "^3.6.1",
 ┊30┊32┊    "moment": "^2.24.0"
 ┊31┊33┊  }
 ┊32┊34┊}
```

##### Changed schema&#x2F;resolvers.ts
```diff
@@ -1,5 +1,65 @@
 ┊ 1┊ 1┊import { IResolvers } from 'apollo-server-express';
+┊  ┊ 2┊import { GraphQLDateTime } from 'graphql-iso-date';
+┊  ┊ 3┊import { ChatDb, db, MessageDb, RecipientDb, UserDb } from "../db";
+┊  ┊ 4┊
+┊  ┊ 5┊let users = db.users;
+┊  ┊ 6┊let chats = db.chats;
+┊  ┊ 7┊const currentUser = users[0];
 ┊ 2┊ 8┊
 ┊ 3┊ 9┊export const resolvers: IResolvers = {
-┊ 4┊  ┊  Query: {},
+┊  ┊10┊  Date: GraphQLDateTime,
+┊  ┊11┊  Query: {
+┊  ┊12┊    me: (): UserDb => currentUser,
+┊  ┊13┊    users: (): UserDb[] => users.filter(user => user.id !== currentUser.id),
+┊  ┊14┊    chats: (): ChatDb[] => chats.filter(chat => chat.listingMemberIds.includes(currentUser.id)),
+┊  ┊15┊    chat: (obj: any, {chatId}): ChatDb | null => chats.find(chat => chat.id === chatId) || null,
+┊  ┊16┊  },
+┊  ┊17┊  Chat: {
+┊  ┊18┊    name: (chat: ChatDb): string => chat.name ? chat.name : users
+┊  ┊19┊      .find(user => user.id === chat.allTimeMemberIds.find(userId => userId !== currentUser.id))!.name,
+┊  ┊20┊    picture: (chat: ChatDb) => chat.name ? chat.picture : users
+┊  ┊21┊      .find(user => user.id === chat.allTimeMemberIds.find(userId => userId !== currentUser.id))!.picture,
+┊  ┊22┊    allTimeMembers: (chat: ChatDb): UserDb[] => users.filter(user => chat.allTimeMemberIds.includes(user.id)),
+┊  ┊23┊    listingMembers: (chat: ChatDb): UserDb[] => users.filter(user => chat.listingMemberIds.includes(user.id)),
+┊  ┊24┊    actualGroupMembers: (chat: ChatDb): UserDb[] => users.filter(user => chat.actualGroupMemberIds && chat.actualGroupMemberIds.includes(user.id)),
+┊  ┊25┊    admins: (chat: ChatDb): UserDb[] => users.filter(user => chat.adminIds && chat.adminIds.includes(user.id)),
+┊  ┊26┊    owner: (chat: ChatDb): UserDb | null => users.find(user => chat.ownerId === user.id) || null,
+┊  ┊27┊    isGroup: (chat: ChatDb): boolean => !!chat.name,
+┊  ┊28┊    messages: (chat: ChatDb, {amount = 0}: {amount: number}): MessageDb[] => {
+┊  ┊29┊      const messages = chat.messages
+┊  ┊30┊      .filter(message => message.holderIds.includes(currentUser.id))
+┊  ┊31┊      .sort((a, b) => b.createdAt.valueOf() - a.createdAt.valueOf()) || <MessageDb[]>[];
+┊  ┊32┊      return (amount ? messages.slice(0, amount) : messages).reverse();
+┊  ┊33┊    },
+┊  ┊34┊    lastMessage: (chat: ChatDb): MessageDb => {
+┊  ┊35┊      return chat.messages
+┊  ┊36┊        .filter(message => message.holderIds.includes(currentUser.id))
+┊  ┊37┊        .sort((a, b) => b.createdAt.valueOf() - a.createdAt.valueOf())[0] || null;
+┊  ┊38┊    },
+┊  ┊39┊    updatedAt: (chat: ChatDb): Date => {
+┊  ┊40┊      const lastMessage = chat.messages
+┊  ┊41┊        .filter(message => message.holderIds.includes(currentUser.id))
+┊  ┊42┊        .sort((a, b) => b.createdAt.valueOf() - a.createdAt.valueOf())[0];
+┊  ┊43┊
+┊  ┊44┊      return lastMessage ? lastMessage.createdAt : chat.createdAt;
+┊  ┊45┊    },
+┊  ┊46┊    unreadMessages: (chat: ChatDb): number => chat.messages
+┊  ┊47┊      .filter(message => message.holderIds.includes(currentUser.id) &&
+┊  ┊48┊        message.recipients.find(recipient => recipient.userId === currentUser.id && !recipient.readAt))
+┊  ┊49┊      .length,
+┊  ┊50┊  },
+┊  ┊51┊  Message: {
+┊  ┊52┊    chat: (message: MessageDb): ChatDb | null => chats.find(chat => message.chatId === chat.id) || null,
+┊  ┊53┊    sender: (message: MessageDb): UserDb | null => users.find(user => user.id === message.senderId) || null,
+┊  ┊54┊    holders: (message: MessageDb): UserDb[] => users.filter(user => message.holderIds.includes(user.id)),
+┊  ┊55┊    ownership: (message: MessageDb): boolean => message.senderId === currentUser.id,
+┊  ┊56┊  },
+┊  ┊57┊  Recipient: {
+┊  ┊58┊    user: (recipient: RecipientDb): UserDb | null => users.find(user => recipient.userId === user.id) || null,
+┊  ┊59┊    message: (recipient: RecipientDb): MessageDb | null => {
+┊  ┊60┊      const chat = chats.find(chat => recipient.chatId === chat.id);
+┊  ┊61┊      return chat ? chat.messages.find(message => recipient.messageId === message.id) || null : null;
+┊  ┊62┊    },
+┊  ┊63┊    chat: (recipient: RecipientDb): ChatDb | null => chats.find(chat => recipient.chatId === chat.id) || null,
+┊  ┊64┊  },
 ┊ 5┊65┊};
```

##### Changed schema&#x2F;typeDefs.ts
```diff
@@ -1,2 +1,76 @@
 ┊ 1┊ 1┊export default `
+┊  ┊ 2┊  scalar Date
+┊  ┊ 3┊
+┊  ┊ 4┊  type Query {
+┊  ┊ 5┊    me: User
+┊  ┊ 6┊    users: [User!]
+┊  ┊ 7┊    chats: [Chat!]!
+┊  ┊ 8┊    chat(chatId: ID!): Chat
+┊  ┊ 9┊  }
+┊  ┊10┊
+┊  ┊11┊  enum MessageType {
+┊  ┊12┊    LOCATION
+┊  ┊13┊    TEXT
+┊  ┊14┊    PICTURE
+┊  ┊15┊  }
+┊  ┊16┊
+┊  ┊17┊  type Chat {
+┊  ┊18┊    #May be a chat or a group
+┊  ┊19┊    id: ID!
+┊  ┊20┊    createdAt: Date!
+┊  ┊21┊    #Computed for chats
+┊  ┊22┊    name: String
+┊  ┊23┊    #Computed for chats
+┊  ┊24┊    picture: String
+┊  ┊25┊    #All members, current and past ones. Includes users who still didn't get the chat listed.
+┊  ┊26┊    allTimeMembers: [User!]!
+┊  ┊27┊    #Whoever gets the chat listed. For groups includes past members who still didn't delete the group. For chats they are the only ones who can send messages.
+┊  ┊28┊    listingMembers: [User!]!
+┊  ┊29┊    #Actual members of the group. Null for chats. For groups they are the only ones who can send messages. They aren't the only ones who get the group listed.
+┊  ┊30┊    actualGroupMembers: [User!]
+┊  ┊31┊    #Null for chats
+┊  ┊32┊    admins: [User!]
+┊  ┊33┊    #If null the group is read-only. Null for chats.
+┊  ┊34┊    owner: User
+┊  ┊35┊    #Computed property
+┊  ┊36┊    isGroup: Boolean!
+┊  ┊37┊    messages(amount: Int): [Message]!
+┊  ┊38┊    #Computed property
+┊  ┊39┊    lastMessage: Message
+┊  ┊40┊    #Computed property
+┊  ┊41┊    updatedAt: Date!
+┊  ┊42┊    #Computed property
+┊  ┊43┊    unreadMessages: Int!
+┊  ┊44┊  }
+┊  ┊45┊
+┊  ┊46┊  type Message {
+┊  ┊47┊    id: ID!
+┊  ┊48┊    sender: User!
+┊  ┊49┊    chat: Chat!
+┊  ┊50┊    content: String!
+┊  ┊51┊    createdAt: Date!
+┊  ┊52┊    #FIXME: should return MessageType
+┊  ┊53┊    type: Int!
+┊  ┊54┊    #Whoever still holds a copy of the message. Cannot be null because the message gets deleted otherwise
+┊  ┊55┊    holders: [User!]!
+┊  ┊56┊    #Computed property
+┊  ┊57┊    ownership: Boolean!
+┊  ┊58┊    #Whoever received the message
+┊  ┊59┊    recipients: [Recipient!]!
+┊  ┊60┊  }
+┊  ┊61┊
+┊  ┊62┊  type Recipient {
+┊  ┊63┊    user: User!
+┊  ┊64┊    message: Message!
+┊  ┊65┊    chat: Chat!
+┊  ┊66┊    receivedAt: Date
+┊  ┊67┊    readAt: Date
+┊  ┊68┊  }
+┊  ┊69┊
+┊  ┊70┊  type User {
+┊  ┊71┊    id: ID!
+┊  ┊72┊    name: String
+┊  ┊73┊    picture: String
+┊  ┊74┊    phone: String
+┊  ┊75┊  }
 ┊ 2┊76┊`;
```

##### Changed yarn.lock
```diff
@@ -118,7 +118,14 @@
 ┊118┊118┊    "@types/express-serve-static-core" "*"
 ┊119┊119┊    "@types/serve-static" "*"
 ┊120┊120┊
-┊121┊   ┊"@types/graphql@^14.0.7":
+┊   ┊121┊"@types/graphql-iso-date@^3.3.1":
+┊   ┊122┊  version "3.3.1"
+┊   ┊123┊  resolved "https://registry.yarnpkg.com/@types/graphql-iso-date/-/graphql-iso-date-3.3.1.tgz#dbb540ae62c68c00eba1d1feb4602d7209257e9d"
+┊   ┊124┊  integrity sha512-x64IejUTqiiC2NGMgMYVOsKgViKknfnhzJHS8pMiYdDqbR5Fd9XHAkujGYvAOBkjFB6TDunY6S8uLDT/OnrKBA==
+┊   ┊125┊  dependencies:
+┊   ┊126┊    "@types/graphql" "*"
+┊   ┊127┊
+┊   ┊128┊"@types/graphql@*", "@types/graphql@^14.0.7":
 ┊122┊129┊  version "14.0.7"
 ┊123┊130┊  resolved "https://registry.yarnpkg.com/@types/graphql/-/graphql-14.0.7.tgz#daa09397220a68ce1cbb3f76a315ff3cd92312f6"
 ┊124┊131┊  integrity sha512-BoLDjdvLQsXPZLJux3lEZANwGr3Xag56Ngy0U3y8uoRSDdeLcn43H3oBcgZlnd++iOQElBpaRVDHPzEDekyvXQ==
```
```diff
@@ -1124,6 +1131,11 @@
 ┊1124┊1131┊  dependencies:
 ┊1125┊1132┊    "@apollographql/apollo-tools" "^0.3.3"
 ┊1126┊1133┊
+┊    ┊1134┊graphql-iso-date@^3.6.1:
+┊    ┊1135┊  version "3.6.1"
+┊    ┊1136┊  resolved "https://registry.yarnpkg.com/graphql-iso-date/-/graphql-iso-date-3.6.1.tgz#bd2d0dc886e0f954cbbbc496bbf1d480b57ffa96"
+┊    ┊1137┊  integrity sha512-AwFGIuYMJQXOEAgRlJlFL4H1ncFM8n8XmoVDTNypNOZyQ8LFDG2ppMFlsS862BSTCDcSUfHp8PD3/uJhv7t59Q==
+┊    ┊1138┊
 ┊1127┊1139┊graphql-subscriptions@^1.0.0:
 ┊1128┊1140┊  version "1.0.0"
 ┊1129┊1141┊  resolved "https://registry.yarnpkg.com/graphql-subscriptions/-/graphql-subscriptions-1.0.0.tgz#475267694b3bd465af6477dbab4263a3f62702b8"
```

[}]: #

# Chapter 6

First, let's install `graphql-code-generator`  in our server and add it to the run scripts:

    $ npm install graphql-code-generator

[{]: <helper> (diffStep "2.1")

#### [Step 2.1: Install graphql-code-generator](https://github.com/Urigo/WhatsApp-Clone-Server/commit/c8572fa)

##### Added codegen.yml
```diff
@@ -0,0 +1,18 @@
+┊  ┊ 1┊overwrite: true
+┊  ┊ 2┊schema: "./schema/typeDefs.ts"
+┊  ┊ 3┊documents: null
+┊  ┊ 4┊require:
+┊  ┊ 5┊  - ts-node/register
+┊  ┊ 6┊generates:
+┊  ┊ 7┊  ./types.d.ts:
+┊  ┊ 8┊    config:
+┊  ┊ 9┊      optionalType: undefined | null
+┊  ┊10┊      mappers:
+┊  ┊11┊        Chat: ./db#ChatDb
+┊  ┊12┊        Message: ./db#MessageDb
+┊  ┊13┊        Recipient: ./db#RecipientDb
+┊  ┊14┊        User: ./db#UserDb
+┊  ┊15┊    plugins:
+┊  ┊16┊      - "typescript-common"
+┊  ┊17┊      - "typescript-server"
+┊  ┊18┊      - "typescript-resolvers"
```

##### Changed package.json
```diff
@@ -8,8 +8,12 @@
 ┊ 8┊ 8┊  },
 ┊ 9┊ 9┊  "scripts": {
 ┊10┊10┊    "build": "tsc",
-┊11┊  ┊    "start": "ts-node index.ts",
-┊12┊  ┊    "dev": "nodemon --exec yarn start:server -e ts"
+┊  ┊11┊    "generate": "gql-gen",
+┊  ┊12┊    "generate:watch": "nodemon --exec yarn generate -e graphql",
+┊  ┊13┊    "start:server": "ts-node index.ts",
+┊  ┊14┊    "start:server:watch": "nodemon --exec yarn start:server -e ts",
+┊  ┊15┊    "dev": "concurrently \"yarn generate:watch\" \"yarn start:server:watch\"",
+┊  ┊16┊    "start": "yarn generate && yarn start:server"
 ┊13┊17┊  },
 ┊14┊18┊  "devDependencies": {
 ┊15┊19┊    "@types/body-parser": "^1.17.0",
```
```diff
@@ -18,6 +22,10 @@
 ┊18┊22┊    "@types/graphql": "^14.0.7",
 ┊19┊23┊    "@types/graphql-iso-date": "^3.3.1",
 ┊20┊24┊    "@types/node": "^11.9.5",
+┊  ┊25┊    "concurrently": "^4.1.0",
+┊  ┊26┊    "graphql-codegen-typescript-common": "^0.17.0",
+┊  ┊27┊    "graphql-codegen-typescript-resolvers": "^0.17.0",
+┊  ┊28┊    "graphql-codegen-typescript-server": "^0.17.0",
 ┊21┊29┊    "nodemon": "^1.18.10",
 ┊22┊30┊    "ts-node": "^8.0.2",
 ┊23┊31┊    "typescript": "^3.3.3333"
```
```diff
@@ -28,6 +36,7 @@
 ┊28┊36┊    "cors": "^2.8.5",
 ┊29┊37┊    "express": "^4.16.4",
 ┊30┊38┊    "graphql": "^14.1.1",
+┊  ┊39┊    "graphql-code-generator": "^0.17.0",
 ┊31┊40┊    "graphql-iso-date": "^3.6.1",
 ┊32┊41┊    "moment": "^2.24.0"
 ┊33┊42┊  }
```

##### Changed yarn.lock
```diff
@@ -14,6 +14,94 @@
 ┊ 14┊ 14┊  resolved "https://registry.yarnpkg.com/@apollographql/graphql-playground-html/-/graphql-playground-html-1.6.6.tgz#022209e28a2b547dcde15b219f0c50f47aa5beb3"
 ┊ 15┊ 15┊  integrity sha512-lqK94b+caNtmKFs5oUVXlSpN3sm5IXZ+KfhMxOtr0LR2SqErzkoJilitjDvJ1WbjHlxLI7WtCjRmOLdOGJqtMQ==
 ┊ 16┊ 16┊
+┊   ┊ 17┊"@babel/code-frame@^7.0.0":
+┊   ┊ 18┊  version "7.0.0"
+┊   ┊ 19┊  resolved "https://registry.yarnpkg.com/@babel/code-frame/-/code-frame-7.0.0.tgz#06e2ab19bdb535385559aabb5ba59729482800f8"
+┊   ┊ 20┊  integrity sha512-OfC2uemaknXr87bdLUkWog7nYuliM9Ij5HUcajsVcMCpQrcLmtxRbVFTIqmcSkSeYRBFBRxs2FiUqFJDLdiebA==
+┊   ┊ 21┊  dependencies:
+┊   ┊ 22┊    "@babel/highlight" "^7.0.0"
+┊   ┊ 23┊
+┊   ┊ 24┊"@babel/generator@^7.2.2":
+┊   ┊ 25┊  version "7.3.3"
+┊   ┊ 26┊  resolved "https://registry.yarnpkg.com/@babel/generator/-/generator-7.3.3.tgz#185962ade59a52e00ca2bdfcfd1d58e528d4e39e"
+┊   ┊ 27┊  integrity sha512-aEADYwRRZjJyMnKN7llGIlircxTCofm3dtV5pmY6ob18MSIuipHpA2yZWkPlycwu5HJcx/pADS3zssd8eY7/6A==
+┊   ┊ 28┊  dependencies:
+┊   ┊ 29┊    "@babel/types" "^7.3.3"
+┊   ┊ 30┊    jsesc "^2.5.1"
+┊   ┊ 31┊    lodash "^4.17.11"
+┊   ┊ 32┊    source-map "^0.5.0"
+┊   ┊ 33┊    trim-right "^1.0.1"
+┊   ┊ 34┊
+┊   ┊ 35┊"@babel/helper-function-name@^7.1.0":
+┊   ┊ 36┊  version "7.1.0"
+┊   ┊ 37┊  resolved "https://registry.yarnpkg.com/@babel/helper-function-name/-/helper-function-name-7.1.0.tgz#a0ceb01685f73355d4360c1247f582bfafc8ff53"
+┊   ┊ 38┊  integrity sha512-A95XEoCpb3TO+KZzJ4S/5uW5fNe26DjBGqf1o9ucyLyCmi1dXq/B3c8iaWTfBk3VvetUxl16e8tIrd5teOCfGw==
+┊   ┊ 39┊  dependencies:
+┊   ┊ 40┊    "@babel/helper-get-function-arity" "^7.0.0"
+┊   ┊ 41┊    "@babel/template" "^7.1.0"
+┊   ┊ 42┊    "@babel/types" "^7.0.0"
+┊   ┊ 43┊
+┊   ┊ 44┊"@babel/helper-get-function-arity@^7.0.0":
+┊   ┊ 45┊  version "7.0.0"
+┊   ┊ 46┊  resolved "https://registry.yarnpkg.com/@babel/helper-get-function-arity/-/helper-get-function-arity-7.0.0.tgz#83572d4320e2a4657263734113c42868b64e49c3"
+┊   ┊ 47┊  integrity sha512-r2DbJeg4svYvt3HOS74U4eWKsUAMRH01Z1ds1zx8KNTPtpTL5JAsdFv8BNyOpVqdFhHkkRDIg5B4AsxmkjAlmQ==
+┊   ┊ 48┊  dependencies:
+┊   ┊ 49┊    "@babel/types" "^7.0.0"
+┊   ┊ 50┊
+┊   ┊ 51┊"@babel/helper-split-export-declaration@^7.0.0":
+┊   ┊ 52┊  version "7.0.0"
+┊   ┊ 53┊  resolved "https://registry.yarnpkg.com/@babel/helper-split-export-declaration/-/helper-split-export-declaration-7.0.0.tgz#3aae285c0311c2ab095d997b8c9a94cad547d813"
+┊   ┊ 54┊  integrity sha512-MXkOJqva62dfC0w85mEf/LucPPS/1+04nmmRMPEBUB++hiiThQ2zPtX/mEWQ3mtzCEjIJvPY8nuwxXtQeQwUag==
+┊   ┊ 55┊  dependencies:
+┊   ┊ 56┊    "@babel/types" "^7.0.0"
+┊   ┊ 57┊
+┊   ┊ 58┊"@babel/highlight@^7.0.0":
+┊   ┊ 59┊  version "7.0.0"
+┊   ┊ 60┊  resolved "https://registry.yarnpkg.com/@babel/highlight/-/highlight-7.0.0.tgz#f710c38c8d458e6dd9a201afb637fcb781ce99e4"
+┊   ┊ 61┊  integrity sha512-UFMC4ZeFC48Tpvj7C8UgLvtkaUuovQX+5xNWrsIoMG8o2z+XFKjKaN9iVmS84dPwVN00W4wPmqvYoZF3EGAsfw==
+┊   ┊ 62┊  dependencies:
+┊   ┊ 63┊    chalk "^2.0.0"
+┊   ┊ 64┊    esutils "^2.0.2"
+┊   ┊ 65┊    js-tokens "^4.0.0"
+┊   ┊ 66┊
+┊   ┊ 67┊"@babel/parser@^7.2.0", "@babel/parser@^7.2.2", "@babel/parser@^7.2.3":
+┊   ┊ 68┊  version "7.3.3"
+┊   ┊ 69┊  resolved "https://registry.yarnpkg.com/@babel/parser/-/parser-7.3.3.tgz#092d450db02bdb6ccb1ca8ffd47d8774a91aef87"
+┊   ┊ 70┊  integrity sha512-xsH1CJoln2r74hR+y7cg2B5JCPaTh+Hd+EbBRk9nWGSNspuo6krjhX0Om6RnRQuIvFq8wVXCLKH3kwKDYhanSg==
+┊   ┊ 71┊
+┊   ┊ 72┊"@babel/template@^7.1.0":
+┊   ┊ 73┊  version "7.2.2"
+┊   ┊ 74┊  resolved "https://registry.yarnpkg.com/@babel/template/-/template-7.2.2.tgz#005b3fdf0ed96e88041330379e0da9a708eb2907"
+┊   ┊ 75┊  integrity sha512-zRL0IMM02AUDwghf5LMSSDEz7sBCO2YnNmpg3uWTZj/v1rcG2BmQUvaGU8GhU8BvfMh1k2KIAYZ7Ji9KXPUg7g==
+┊   ┊ 76┊  dependencies:
+┊   ┊ 77┊    "@babel/code-frame" "^7.0.0"
+┊   ┊ 78┊    "@babel/parser" "^7.2.2"
+┊   ┊ 79┊    "@babel/types" "^7.2.2"
+┊   ┊ 80┊
+┊   ┊ 81┊"@babel/traverse@^7.1.6":
+┊   ┊ 82┊  version "7.2.3"
+┊   ┊ 83┊  resolved "https://registry.yarnpkg.com/@babel/traverse/-/traverse-7.2.3.tgz#7ff50cefa9c7c0bd2d81231fdac122f3957748d8"
+┊   ┊ 84┊  integrity sha512-Z31oUD/fJvEWVR0lNZtfgvVt512ForCTNKYcJBGbPb1QZfve4WGH8Wsy7+Mev33/45fhP/hwQtvgusNdcCMgSw==
+┊   ┊ 85┊  dependencies:
+┊   ┊ 86┊    "@babel/code-frame" "^7.0.0"
+┊   ┊ 87┊    "@babel/generator" "^7.2.2"
+┊   ┊ 88┊    "@babel/helper-function-name" "^7.1.0"
+┊   ┊ 89┊    "@babel/helper-split-export-declaration" "^7.0.0"
+┊   ┊ 90┊    "@babel/parser" "^7.2.3"
+┊   ┊ 91┊    "@babel/types" "^7.2.2"
+┊   ┊ 92┊    debug "^4.1.0"
+┊   ┊ 93┊    globals "^11.1.0"
+┊   ┊ 94┊    lodash "^4.17.10"
+┊   ┊ 95┊
+┊   ┊ 96┊"@babel/types@^7.0.0", "@babel/types@^7.2.0", "@babel/types@^7.2.2", "@babel/types@^7.3.3":
+┊   ┊ 97┊  version "7.3.3"
+┊   ┊ 98┊  resolved "https://registry.yarnpkg.com/@babel/types/-/types-7.3.3.tgz#6c44d1cdac2a7625b624216657d5bc6c107ab436"
+┊   ┊ 99┊  integrity sha512-2tACZ80Wg09UnPg5uGAOUvvInaqLk3l/IAhQzlxLQOIXacr6bMsra5SH6AWw/hIDRCSbCdHP2KzSOD+cT7TzMQ==
+┊   ┊100┊  dependencies:
+┊   ┊101┊    esutils "^2.0.2"
+┊   ┊102┊    lodash "^4.17.11"
+┊   ┊103┊    to-fast-properties "^2.0.0"
+┊   ┊104┊
 ┊ 17┊105┊"@protobufjs/aspromise@^1.1.1", "@protobufjs/aspromise@^1.1.2":
 ┊ 18┊106┊  version "1.1.2"
 ┊ 19┊107┊  resolved "https://registry.yarnpkg.com/@protobufjs/aspromise/-/aspromise-1.1.2.tgz#9b8b0cc663d669a7d8f6f5d0893a14d348f30fbf"
```
```diff
@@ -67,6 +155,13 @@
 ┊ 67┊155┊  resolved "https://registry.yarnpkg.com/@protobufjs/utf8/-/utf8-1.1.0.tgz#a777360b5b39a1a2e5106f8e858f2fd2d060c570"
 ┊ 68┊156┊  integrity sha1-p3c2C1s5oaLlEG+OhY8v0tBgxXA=
 ┊ 69┊157┊
+┊   ┊158┊"@samverschueren/stream-to-observable@^0.3.0":
+┊   ┊159┊  version "0.3.0"
+┊   ┊160┊  resolved "https://registry.yarnpkg.com/@samverschueren/stream-to-observable/-/stream-to-observable-0.3.0.tgz#ecdf48d532c58ea477acfcab80348424f8d0662f"
+┊   ┊161┊  integrity sha512-MI4Xx6LHs4Webyvi6EbspgyAb4D2Q2VtnCQ1blOJcoLS6mVa8lNN2rkIy1CVxfTUpoyIbCTkXES1rLXztFD1lg==
+┊   ┊162┊  dependencies:
+┊   ┊163┊    any-observable "^0.3.0"
+┊   ┊164┊
 ┊ 70┊165┊"@types/accepts@^1.3.5":
 ┊ 71┊166┊  version "1.3.5"
 ┊ 72┊167┊  resolved "https://registry.yarnpkg.com/@types/accepts/-/accepts-1.3.5.tgz#c34bec115cfc746e04fe5a059df4ce7e7b391575"
```
```diff
@@ -74,6 +169,18 @@
 ┊ 74┊169┊  dependencies:
 ┊ 75┊170┊    "@types/node" "*"
 ┊ 76┊171┊
+┊   ┊172┊"@types/babel-types@*":
+┊   ┊173┊  version "7.0.5"
+┊   ┊174┊  resolved "https://registry.yarnpkg.com/@types/babel-types/-/babel-types-7.0.5.tgz#26f5bba8c58acd9b84d1a9135fb2789a1c191bc1"
+┊   ┊175┊  integrity sha512-0t0R7fKAXT/P++S98djRkXbL9Sxd9NNtfNg3BNw2EQOjVIkiMBdmO55N2Tp3wGK3mylmM7Vck9h5tEoSuSUabA==
+┊   ┊176┊
+┊   ┊177┊"@types/babylon@6.16.5":
+┊   ┊178┊  version "6.16.5"
+┊   ┊179┊  resolved "https://registry.yarnpkg.com/@types/babylon/-/babylon-6.16.5.tgz#1c5641db69eb8cdf378edd25b4be7754beeb48b4"
+┊   ┊180┊  integrity sha512-xH2e58elpj1X4ynnKp9qSnWlsRTIs6n3tgLGNfwAGHwePw0mulHQllV34n0T25uYSu1k0hRKkWXF890B1yS47w==
+┊   ┊181┊  dependencies:
+┊   ┊182┊    "@types/babel-types" "*"
+┊   ┊183┊
 ┊ 77┊184┊"@types/body-parser@*", "@types/body-parser@1.17.0", "@types/body-parser@^1.17.0":
 ┊ 78┊185┊  version "1.17.0"
 ┊ 79┊186┊  resolved "https://registry.yarnpkg.com/@types/body-parser/-/body-parser-1.17.0.tgz#9f5c9d9bd04bb54be32d5eb9fc0d8c974e6cf58c"
```
```diff
@@ -130,6 +237,11 @@
 ┊130┊237┊  resolved "https://registry.yarnpkg.com/@types/graphql/-/graphql-14.0.7.tgz#daa09397220a68ce1cbb3f76a315ff3cd92312f6"
 ┊131┊238┊  integrity sha512-BoLDjdvLQsXPZLJux3lEZANwGr3Xag56Ngy0U3y8uoRSDdeLcn43H3oBcgZlnd++iOQElBpaRVDHPzEDekyvXQ==
 ┊132┊239┊
+┊   ┊240┊"@types/is-glob@4.0.0":
+┊   ┊241┊  version "4.0.0"
+┊   ┊242┊  resolved "https://registry.yarnpkg.com/@types/is-glob/-/is-glob-4.0.0.tgz#fb8a2bff539025d4dcd6d5efe7689e03341b876d"
+┊   ┊243┊  integrity sha512-zC/2EmD8scdsGIeE+Xg7kP7oi9VP90zgMQtm9Cr25av4V+a+k8slQyiT60qSw8KORYrOKlPXfHwoa1bQbRzskQ==
+┊   ┊244┊
 ┊133┊245┊"@types/long@^4.0.0":
 ┊134┊246┊  version "4.0.0"
 ┊135┊247┊  resolved "https://registry.yarnpkg.com/@types/long/-/long-4.0.0.tgz#719551d2352d301ac8b81db732acb6bdc28dbdef"
```
```diff
@@ -150,6 +262,11 @@
 ┊150┊262┊  resolved "https://registry.yarnpkg.com/@types/node/-/node-10.12.27.tgz#eb3843f15d0ba0986cc7e4d734d2ee8b50709ef8"
 ┊151┊263┊  integrity sha512-e9wgeY6gaY21on3ve0xAjgBVjGDWq/xUteK0ujsE53bUoxycMkqfnkUgMt6ffZtykZ5X12Mg3T7Pw4TRCObDKg==
 ┊152┊264┊
+┊   ┊265┊"@types/prettier@1.16.1":
+┊   ┊266┊  version "1.16.1"
+┊   ┊267┊  resolved "https://registry.yarnpkg.com/@types/prettier/-/prettier-1.16.1.tgz#328d1c9b54402e44119398bcb6a31b7bbd606d59"
+┊   ┊268┊  integrity sha512-db6pZL5QY3JrlCHBhYQzYDci0xnoDuxfseUuguLRr3JNk+bnCfpkK6p8quiUDyO8A0vbpBKkk59Fw125etrNeA==
+┊   ┊269┊
 ┊153┊270┊"@types/range-parser@*":
 ┊154┊271┊  version "1.2.3"
 ┊155┊272┊  resolved "https://registry.yarnpkg.com/@types/range-parser/-/range-parser-1.2.3.tgz#7ee330ba7caafb98090bece86a5ee44115904c2c"
```
```diff
@@ -163,6 +280,11 @@
 ┊163┊280┊    "@types/express-serve-static-core" "*"
 ┊164┊281┊    "@types/mime" "*"
 ┊165┊282┊
+┊   ┊283┊"@types/valid-url@1.0.2":
+┊   ┊284┊  version "1.0.2"
+┊   ┊285┊  resolved "https://registry.yarnpkg.com/@types/valid-url/-/valid-url-1.0.2.tgz#60fa435ce24bfd5ba107b8d2a80796aeaf3a8f45"
+┊   ┊286┊  integrity sha1-YPpDXOJL/VuhB7jSqAeWrq86j0U=
+┊   ┊287┊
 ┊166┊288┊"@types/ws@^6.0.0":
 ┊167┊289┊  version "6.0.1"
 ┊168┊290┊  resolved "https://registry.yarnpkg.com/@types/ws/-/ws-6.0.1.tgz#ca7a3f3756aa12f62a0a62145ed14c6db25d5a28"
```
```diff
@@ -184,6 +306,16 @@
 ┊184┊306┊    mime-types "~2.1.18"
 ┊185┊307┊    negotiator "0.6.1"
 ┊186┊308┊
+┊   ┊309┊ajv@^6.5.5:
+┊   ┊310┊  version "6.9.2"
+┊   ┊311┊  resolved "https://registry.yarnpkg.com/ajv/-/ajv-6.9.2.tgz#4927adb83e7f48e5a32b45729744c71ec39c9c7b"
+┊   ┊312┊  integrity sha512-4UFy0/LgDo7Oa/+wOAlj44tp9K78u38E5/359eSrqEp1Z5PdVfimCcs7SluXMP755RUQu6d2b4AvF0R1C9RZjg==
+┊   ┊313┊  dependencies:
+┊   ┊314┊    fast-deep-equal "^2.0.1"
+┊   ┊315┊    fast-json-stable-stringify "^2.0.0"
+┊   ┊316┊    json-schema-traverse "^0.4.1"
+┊   ┊317┊    uri-js "^4.2.2"
+┊   ┊318┊
 ┊187┊319┊ansi-align@^2.0.0:
 ┊188┊320┊  version "2.0.0"
 ┊189┊321┊  resolved "https://registry.yarnpkg.com/ansi-align/-/ansi-align-2.0.0.tgz#c36aeccba563b89ceb556f3690f0b1d9e3547f7f"
```
```diff
@@ -191,6 +323,11 @@
 ┊191┊323┊  dependencies:
 ┊192┊324┊    string-width "^2.0.0"
 ┊193┊325┊
+┊   ┊326┊ansi-escapes@^3.0.0, ansi-escapes@^3.2.0:
+┊   ┊327┊  version "3.2.0"
+┊   ┊328┊  resolved "https://registry.yarnpkg.com/ansi-escapes/-/ansi-escapes-3.2.0.tgz#8780b98ff9dbf5638152d1f1fe5c1d7b4442976b"
+┊   ┊329┊  integrity sha512-cBhpre4ma+U0T1oM5fXg7Dy1Jw7zzwv7lt/GoCpr+hDQJoYnKVPLL4dCvSEFMmQurOQvSrwT7SL/DAlhBI97RQ==
+┊   ┊330┊
 ┊194┊331┊ansi-regex@^2.0.0:
 ┊195┊332┊  version "2.1.1"
 ┊196┊333┊  resolved "https://registry.yarnpkg.com/ansi-regex/-/ansi-regex-2.1.1.tgz#c3b33ab5ee360d86e0e628f0468ae7ef27d654df"
```
```diff
@@ -201,6 +338,16 @@
 ┊201┊338┊  resolved "https://registry.yarnpkg.com/ansi-regex/-/ansi-regex-3.0.0.tgz#ed0317c322064f79466c02966bddb605ab37d998"
 ┊202┊339┊  integrity sha1-7QMXwyIGT3lGbAKWa922Bas32Zg=
 ┊203┊340┊
+┊   ┊341┊ansi-regex@^4.0.0:
+┊   ┊342┊  version "4.0.0"
+┊   ┊343┊  resolved "https://registry.yarnpkg.com/ansi-regex/-/ansi-regex-4.0.0.tgz#70de791edf021404c3fd615aa89118ae0432e5a9"
+┊   ┊344┊  integrity sha512-iB5Dda8t/UqpPI/IjsejXu5jOGDrzn41wJyljwPH65VCIbk6+1BzFIMJGFwTNrYXT1CrD+B4l19U7awiQ8rk7w==
+┊   ┊345┊
+┊   ┊346┊ansi-styles@^2.2.1:
+┊   ┊347┊  version "2.2.1"
+┊   ┊348┊  resolved "https://registry.yarnpkg.com/ansi-styles/-/ansi-styles-2.2.1.tgz#b432dd3358b634cf75e1e4664368240533c1ddbe"
+┊   ┊349┊  integrity sha1-tDLdM1i2NM914eRmQ2gkBTPB3b4=
+┊   ┊350┊
 ┊204┊351┊ansi-styles@^3.2.1:
 ┊205┊352┊  version "3.2.1"
 ┊206┊353┊  resolved "https://registry.yarnpkg.com/ansi-styles/-/ansi-styles-3.2.1.tgz#41fbb20243e50b12be0f04b8dedbf07520ce841d"
```
```diff
@@ -208,6 +355,11 @@
 ┊208┊355┊  dependencies:
 ┊209┊356┊    color-convert "^1.9.0"
 ┊210┊357┊
+┊   ┊358┊any-observable@^0.3.0:
+┊   ┊359┊  version "0.3.0"
+┊   ┊360┊  resolved "https://registry.yarnpkg.com/any-observable/-/any-observable-0.3.0.tgz#af933475e5806a67d0d7df090dd5e8bef65d119b"
+┊   ┊361┊  integrity sha512-/FQM1EDkTsf63Ub2C6O7GuYFDsSXUwsaZDurV0np41ocwq0jthUAYCmhBX9f+KwlaCgIuWyr/4WlUQUBfKfZog==
+┊   ┊362┊
 ┊211┊363┊anymatch@^2.0.0:
 ┊212┊364┊  version "2.0.0"
 ┊213┊365┊  resolved "https://registry.yarnpkg.com/anymatch/-/anymatch-2.0.0.tgz#bcb24b4f37934d9aa7ac17b4adaf89e7c76ef2eb"
```
```diff
@@ -376,6 +528,13 @@
 ┊376┊528┊  resolved "https://registry.yarnpkg.com/arg/-/arg-4.1.0.tgz#583c518199419e0037abb74062c37f8519e575f0"
 ┊377┊529┊  integrity sha512-ZWc51jO3qegGkVh8Hwpv636EkbesNV5ZNQPCtRa+0qytRYPEs9IYT9qITY9buezqUH5uqyzlWLcufrzU2rffdg==
 ┊378┊530┊
+┊   ┊531┊argparse@^1.0.7:
+┊   ┊532┊  version "1.0.10"
+┊   ┊533┊  resolved "https://registry.yarnpkg.com/argparse/-/argparse-1.0.10.tgz#bcd6791ea5ae09725e17e5ad988134cd40b3d911"
+┊   ┊534┊  integrity sha512-o5Roy6tNG4SL/FOkCAN6RzjiakZS25RLYFrcMttJqbdd8BWrnA+fGz57iN5Pb06pvBGvl5gQ0B48dJlslXvoTg==
+┊   ┊535┊  dependencies:
+┊   ┊536┊    sprintf-js "~1.0.2"
+┊   ┊537┊
 ┊379┊538┊arr-diff@^4.0.0:
 ┊380┊539┊  version "4.0.0"
 ┊381┊540┊  resolved "https://registry.yarnpkg.com/arr-diff/-/arr-diff-4.0.0.tgz#d6461074febfec71e7e15235761a329a5dc7c520"
```
```diff
@@ -401,6 +560,18 @@
 ┊401┊560┊  resolved "https://registry.yarnpkg.com/array-unique/-/array-unique-0.3.2.tgz#a894b75d4bc4f6cd679ef3244a9fd8f46ae2d428"
 ┊402┊561┊  integrity sha1-qJS3XUvE9s1nnvMkSp/Y9Gri1Cg=
 ┊403┊562┊
+┊   ┊563┊asn1@~0.2.3:
+┊   ┊564┊  version "0.2.4"
+┊   ┊565┊  resolved "https://registry.yarnpkg.com/asn1/-/asn1-0.2.4.tgz#8d2475dfab553bb33e77b54e59e880bb8ce23136"
+┊   ┊566┊  integrity sha512-jxwzQpLQjSmWXgwaCZE9Nz+glAG01yF1QnWgbhGwHI5A6FRIEY6IVqtHhIepHqI7/kyEyQEagBC5mBEFlIYvdg==
+┊   ┊567┊  dependencies:
+┊   ┊568┊    safer-buffer "~2.1.0"
+┊   ┊569┊
+┊   ┊570┊assert-plus@1.0.0, assert-plus@^1.0.0:
+┊   ┊571┊  version "1.0.0"
+┊   ┊572┊  resolved "https://registry.yarnpkg.com/assert-plus/-/assert-plus-1.0.0.tgz#f12e0f3c5d77b0b1cdd9146942e4e96c1e4dd525"
+┊   ┊573┊  integrity sha1-8S4PPF13sLHN2RRpQuTpbB5N1SU=
+┊   ┊574┊
 ┊404┊575┊assign-symbols@^1.0.0:
 ┊405┊576┊  version "1.0.0"
 ┊406┊577┊  resolved "https://registry.yarnpkg.com/assign-symbols/-/assign-symbols-1.0.0.tgz#59667f41fadd4f20ccbc2bb96b8d4f7f78ec0367"
```
```diff
@@ -423,11 +594,47 @@
 ┊423┊594┊  dependencies:
 ┊424┊595┊    retry "0.12.0"
 ┊425┊596┊
+┊   ┊597┊async@^2.6.1:
+┊   ┊598┊  version "2.6.2"
+┊   ┊599┊  resolved "https://registry.yarnpkg.com/async/-/async-2.6.2.tgz#18330ea7e6e313887f5d2f2a904bac6fe4dd5381"
+┊   ┊600┊  integrity sha512-H1qVYh1MYhEEFLsP97cVKqCGo7KfCyTt6uEWqsTBr9SO84oK9Uwbyd/yCW+6rKJLHksBNUVWZDAjfS+Ccx0Bbg==
+┊   ┊601┊  dependencies:
+┊   ┊602┊    lodash "^4.17.11"
+┊   ┊603┊
+┊   ┊604┊asynckit@^0.4.0:
+┊   ┊605┊  version "0.4.0"
+┊   ┊606┊  resolved "https://registry.yarnpkg.com/asynckit/-/asynckit-0.4.0.tgz#c79ed97f7f34cb8f2ba1bc9790bcc366474b4b79"
+┊   ┊607┊  integrity sha1-x57Zf380y48robyXkLzDZkdLS3k=
+┊   ┊608┊
 ┊426┊609┊atob@^2.1.1:
 ┊427┊610┊  version "2.1.2"
 ┊428┊611┊  resolved "https://registry.yarnpkg.com/atob/-/atob-2.1.2.tgz#6d9517eb9e030d2436666651e86bd9f6f13533c9"
 ┊429┊612┊  integrity sha512-Wm6ukoaOGJi/73p/cl2GvLjTI5JM1k/O14isD73YML8StrH/7/lRFgmg8nICZgD3bZZvjwCGxtMOD3wWNAu8cg==
 ┊430┊613┊
+┊   ┊614┊aws-sign2@~0.7.0:
+┊   ┊615┊  version "0.7.0"
+┊   ┊616┊  resolved "https://registry.yarnpkg.com/aws-sign2/-/aws-sign2-0.7.0.tgz#b46e890934a9591f2d2f6f86d7e6a9f1b3fe76a8"
+┊   ┊617┊  integrity sha1-tG6JCTSpWR8tL2+G1+ap8bP+dqg=
+┊   ┊618┊
+┊   ┊619┊aws4@^1.8.0:
+┊   ┊620┊  version "1.8.0"
+┊   ┊621┊  resolved "https://registry.yarnpkg.com/aws4/-/aws4-1.8.0.tgz#f0e003d9ca9e7f59c7a508945d7b2ef9a04a542f"
+┊   ┊622┊  integrity sha512-ReZxvNHIOv88FlT7rxcXIIC0fPt4KZqZbOlivyWtXLt8ESx84zd3kMC6iK5jVeS2qt+g7ftS7ye4fi06X5rtRQ==
+┊   ┊623┊
+┊   ┊624┊babel-types@7.0.0-beta.3:
+┊   ┊625┊  version "7.0.0-beta.3"
+┊   ┊626┊  resolved "https://registry.yarnpkg.com/babel-types/-/babel-types-7.0.0-beta.3.tgz#cd927ca70e0ae8ab05f4aab83778cfb3e6eb20b4"
+┊   ┊627┊  integrity sha512-36k8J+byAe181OmCMawGhw+DtKO7AwexPVtsPXoMfAkjtZgoCX3bEuHWfdE5sYxRM8dojvtG/+O08M0Z/YDC6w==
+┊   ┊628┊  dependencies:
+┊   ┊629┊    esutils "^2.0.2"
+┊   ┊630┊    lodash "^4.2.0"
+┊   ┊631┊    to-fast-properties "^2.0.0"
+┊   ┊632┊
+┊   ┊633┊babylon@7.0.0-beta.47:
+┊   ┊634┊  version "7.0.0-beta.47"
+┊   ┊635┊  resolved "https://registry.yarnpkg.com/babylon/-/babylon-7.0.0-beta.47.tgz#6d1fa44f0abec41ab7c780481e62fd9aafbdea80"
+┊   ┊636┊  integrity sha512-+rq2cr4GDhtToEzKFD6KZZMDBXhjFAr9JjPw9pAppZACeEWqNM294j+NdBzkSHYXwzzBmVjZ3nEVJlOhbR2gOQ==
+┊   ┊637┊
 ┊431┊638┊backo2@^1.0.2:
 ┊432┊639┊  version "1.0.2"
 ┊433┊640┊  resolved "https://registry.yarnpkg.com/backo2/-/backo2-1.0.2.tgz#31ab1ac8b129363463e35b3ebb69f4dfcfba7947"
```
```diff
@@ -451,6 +658,13 @@
 ┊451┊658┊    mixin-deep "^1.2.0"
 ┊452┊659┊    pascalcase "^0.1.1"
 ┊453┊660┊
+┊   ┊661┊bcrypt-pbkdf@^1.0.0:
+┊   ┊662┊  version "1.0.2"
+┊   ┊663┊  resolved "https://registry.yarnpkg.com/bcrypt-pbkdf/-/bcrypt-pbkdf-1.0.2.tgz#a4301d389b6a43f9b67ff3ca11a3f6637e360e9e"
+┊   ┊664┊  integrity sha1-pDAdOJtqQ/m2f/PKEaP2Y342Dp4=
+┊   ┊665┊  dependencies:
+┊   ┊666┊    tweetnacl "^0.14.3"
+┊   ┊667┊
 ┊454┊668┊binary-extensions@^1.0.0:
 ┊455┊669┊  version "1.13.0"
 ┊456┊670┊  resolved "https://registry.yarnpkg.com/binary-extensions/-/binary-extensions-1.13.0.tgz#9523e001306a32444b907423f1de2164222f6ab1"
```
```diff
@@ -541,17 +755,35 @@
 ┊541┊755┊    union-value "^1.0.0"
 ┊542┊756┊    unset-value "^1.0.0"
 ┊543┊757┊
+┊   ┊758┊camel-case@^3.0.0:
+┊   ┊759┊  version "3.0.0"
+┊   ┊760┊  resolved "https://registry.yarnpkg.com/camel-case/-/camel-case-3.0.0.tgz#ca3c3688a4e9cf3a4cda777dc4dcbc713249cf73"
+┊   ┊761┊  integrity sha1-yjw2iKTpzzpM2nd9xNy8cTJJz3M=
+┊   ┊762┊  dependencies:
+┊   ┊763┊    no-case "^2.2.0"
+┊   ┊764┊    upper-case "^1.1.1"
+┊   ┊765┊
 ┊544┊766┊camelcase@^4.0.0:
 ┊545┊767┊  version "4.1.0"
 ┊546┊768┊  resolved "https://registry.yarnpkg.com/camelcase/-/camelcase-4.1.0.tgz#d545635be1e33c542649c69173e5de6acfae34dd"
 ┊547┊769┊  integrity sha1-1UVjW+HjPFQmScaRc+Xeas+uNN0=
 ┊548┊770┊
+┊   ┊771┊camelcase@^5.0.0:
+┊   ┊772┊  version "5.0.0"
+┊   ┊773┊  resolved "https://registry.yarnpkg.com/camelcase/-/camelcase-5.0.0.tgz#03295527d58bd3cd4aa75363f35b2e8d97be2f42"
+┊   ┊774┊  integrity sha512-faqwZqnWxbxn+F1d399ygeamQNy3lPp/H9H6rNrqYh4FSVCtcY+3cub1MxA8o9mDd55mM8Aghuu/kuyYA6VTsA==
+┊   ┊775┊
 ┊549┊776┊capture-stack-trace@^1.0.0:
 ┊550┊777┊  version "1.0.1"
 ┊551┊778┊  resolved "https://registry.yarnpkg.com/capture-stack-trace/-/capture-stack-trace-1.0.1.tgz#a6c0bbe1f38f3aa0b92238ecb6ff42c344d4135d"
 ┊552┊779┊  integrity sha512-mYQLZnx5Qt1JgB1WEiMCf2647plpGeQ2NMR/5L0HNZzGQo4fuSPnK+wjfPnKZV0aiJDgzmWqqkV/g7JD+DW0qw==
 ┊553┊780┊
-┊554┊   ┊chalk@^2.0.1:
+┊   ┊781┊caseless@~0.12.0:
+┊   ┊782┊  version "0.12.0"
+┊   ┊783┊  resolved "https://registry.yarnpkg.com/caseless/-/caseless-0.12.0.tgz#1b681c21ff84033c826543090689420d187151dc"
+┊   ┊784┊  integrity sha1-G2gcIf+EAzyCZUMJBolCDRhxUdw=
+┊   ┊785┊
+┊   ┊786┊chalk@2.4.2, chalk@^2.0.0, chalk@^2.0.1, chalk@^2.4.1, chalk@^2.4.2:
 ┊555┊787┊  version "2.4.2"
 ┊556┊788┊  resolved "https://registry.yarnpkg.com/chalk/-/chalk-2.4.2.tgz#cd42541677a54333cf541a49108c1432b44c9424"
 ┊557┊789┊  integrity sha512-Mti+f9lpJNcwF4tWV8/OrTTtF1gZi+f8FqlyAdouralcFWFQWF2+NgCHShjkCb+IFBLq9buZwE1xckQU4peSuQ==
```
```diff
@@ -560,7 +792,47 @@
 ┊560┊792┊    escape-string-regexp "^1.0.5"
 ┊561┊793┊    supports-color "^5.3.0"
 ┊562┊794┊
-┊563┊   ┊chokidar@^2.1.0:
+┊   ┊795┊chalk@^1.0.0, chalk@^1.1.3:
+┊   ┊796┊  version "1.1.3"
+┊   ┊797┊  resolved "https://registry.yarnpkg.com/chalk/-/chalk-1.1.3.tgz#a8115c55e4a702fe4d150abd3872822a7e09fc98"
+┊   ┊798┊  integrity sha1-qBFcVeSnAv5NFQq9OHKCKn4J/Jg=
+┊   ┊799┊  dependencies:
+┊   ┊800┊    ansi-styles "^2.2.1"
+┊   ┊801┊    escape-string-regexp "^1.0.2"
+┊   ┊802┊    has-ansi "^2.0.0"
+┊   ┊803┊    strip-ansi "^3.0.0"
+┊   ┊804┊    supports-color "^2.0.0"
+┊   ┊805┊
+┊   ┊806┊change-case@3.1.0:
+┊   ┊807┊  version "3.1.0"
+┊   ┊808┊  resolved "https://registry.yarnpkg.com/change-case/-/change-case-3.1.0.tgz#0e611b7edc9952df2e8513b27b42de72647dd17e"
+┊   ┊809┊  integrity sha512-2AZp7uJZbYEzRPsFoa+ijKdvp9zsrnnt6+yFokfwEpeJm0xuJDVoxiRCAaTzyJND8GJkofo2IcKWaUZ/OECVzw==
+┊   ┊810┊  dependencies:
+┊   ┊811┊    camel-case "^3.0.0"
+┊   ┊812┊    constant-case "^2.0.0"
+┊   ┊813┊    dot-case "^2.1.0"
+┊   ┊814┊    header-case "^1.0.0"
+┊   ┊815┊    is-lower-case "^1.1.0"
+┊   ┊816┊    is-upper-case "^1.1.0"
+┊   ┊817┊    lower-case "^1.1.1"
+┊   ┊818┊    lower-case-first "^1.0.0"
+┊   ┊819┊    no-case "^2.3.2"
+┊   ┊820┊    param-case "^2.1.0"
+┊   ┊821┊    pascal-case "^2.0.0"
+┊   ┊822┊    path-case "^2.1.0"
+┊   ┊823┊    sentence-case "^2.1.0"
+┊   ┊824┊    snake-case "^2.1.0"
+┊   ┊825┊    swap-case "^1.1.0"
+┊   ┊826┊    title-case "^2.1.0"
+┊   ┊827┊    upper-case "^1.1.1"
+┊   ┊828┊    upper-case-first "^1.1.0"
+┊   ┊829┊
+┊   ┊830┊chardet@^0.7.0:
+┊   ┊831┊  version "0.7.0"
+┊   ┊832┊  resolved "https://registry.yarnpkg.com/chardet/-/chardet-0.7.0.tgz#90094849f0937f2eedc2425d0d28a9e5f0cbad9e"
+┊   ┊833┊  integrity sha512-mT8iDcrh03qDGRRmoA2hmBJnxpllMR+0/0qlzjqZES6NdiWDcZkCNAk4rPFZ9Q85r27unkiNNg8ZOiwZXBHwcA==
+┊   ┊834┊
+┊   ┊835┊chokidar@2.1.2, chokidar@^2.1.0:
 ┊564┊836┊  version "2.1.2"
 ┊565┊837┊  resolved "https://registry.yarnpkg.com/chokidar/-/chokidar-2.1.2.tgz#9c23ea40b01638439e0513864d362aeacc5ad058"
 ┊566┊838┊  integrity sha512-IwXUx0FXc5ibYmPC2XeEj5mpXoV66sR+t3jqu2NS2GYwCktt3KF1/Qqjws/NkegajBA4RbZ5+DDwlOiJsxDHEg==
```
```diff
@@ -604,6 +876,35 @@
 ┊604┊876┊  resolved "https://registry.yarnpkg.com/cli-boxes/-/cli-boxes-1.0.0.tgz#4fa917c3e59c94a004cd61f8ee509da651687143"
 ┊605┊877┊  integrity sha1-T6kXw+WclKAEzWH47lCdplFocUM=
 ┊606┊878┊
+┊   ┊879┊cli-cursor@^2.0.0, cli-cursor@^2.1.0:
+┊   ┊880┊  version "2.1.0"
+┊   ┊881┊  resolved "https://registry.yarnpkg.com/cli-cursor/-/cli-cursor-2.1.0.tgz#b35dac376479facc3e94747d41d0d0f5238ffcb5"
+┊   ┊882┊  integrity sha1-s12sN2R5+sw+lHR9QdDQ9SOP/LU=
+┊   ┊883┊  dependencies:
+┊   ┊884┊    restore-cursor "^2.0.0"
+┊   ┊885┊
+┊   ┊886┊cli-truncate@^0.2.1:
+┊   ┊887┊  version "0.2.1"
+┊   ┊888┊  resolved "https://registry.yarnpkg.com/cli-truncate/-/cli-truncate-0.2.1.tgz#9f15cfbb0705005369216c626ac7d05ab90dd574"
+┊   ┊889┊  integrity sha1-nxXPuwcFAFNpIWxiasfQWrkN1XQ=
+┊   ┊890┊  dependencies:
+┊   ┊891┊    slice-ansi "0.0.4"
+┊   ┊892┊    string-width "^1.0.1"
+┊   ┊893┊
+┊   ┊894┊cli-width@^2.0.0:
+┊   ┊895┊  version "2.2.0"
+┊   ┊896┊  resolved "https://registry.yarnpkg.com/cli-width/-/cli-width-2.2.0.tgz#ff19ede8a9a5e579324147b0c11f0fbcbabed639"
+┊   ┊897┊  integrity sha1-/xnt6Kml5XkyQUewwR8PvLq+1jk=
+┊   ┊898┊
+┊   ┊899┊cliui@^4.0.0:
+┊   ┊900┊  version "4.1.0"
+┊   ┊901┊  resolved "https://registry.yarnpkg.com/cliui/-/cliui-4.1.0.tgz#348422dbe82d800b3022eef4f6ac10bf2e4d1b49"
+┊   ┊902┊  integrity sha512-4FG+RSG9DL7uEwRUZXZn3SS34DiDPfzP0VOiEwtUWlE+AR2EIg+hSyvrIgUUfhdgR/UkAeW2QHgeP+hWrXs7jQ==
+┊   ┊903┊  dependencies:
+┊   ┊904┊    string-width "^2.1.1"
+┊   ┊905┊    strip-ansi "^4.0.0"
+┊   ┊906┊    wrap-ansi "^2.0.0"
+┊   ┊907┊
 ┊607┊908┊code-point-at@^1.0.0:
 ┊608┊909┊  version "1.1.0"
 ┊609┊910┊  resolved "https://registry.yarnpkg.com/code-point-at/-/code-point-at-1.1.0.tgz#0d070b4d043a5bea33a2f1a40e2edb3d9a4ccf77"
```
```diff
@@ -617,7 +918,7 @@
 ┊617┊918┊    map-visit "^1.0.0"
 ┊618┊919┊    object-visit "^1.0.0"
 ┊619┊920┊
-┊620┊   ┊color-convert@^1.9.0:
+┊   ┊921┊color-convert@^1.9.0, color-convert@^1.9.1:
 ┊621┊922┊  version "1.9.3"
 ┊622┊923┊  resolved "https://registry.yarnpkg.com/color-convert/-/color-convert-1.9.3.tgz#bb71850690e1f136567de629d2d5471deda4c1e8"
 ┊623┊924┊  integrity sha512-QfAUtd+vFdAtFQcC8CCyYt1fYWxSqAiK2cSD6zDB8N3cpsEBAvRxp9zOGg6G/SHHJYAT88/az/IuDGALsNVbGg==
```
```diff
@@ -629,6 +930,62 @@
 ┊629┊930┊  resolved "https://registry.yarnpkg.com/color-name/-/color-name-1.1.3.tgz#a7d0558bd89c42f795dd42328f740831ca53bc25"
 ┊630┊931┊  integrity sha1-p9BVi9icQveV3UIyj3QIMcpTvCU=
 ┊631┊932┊
+┊   ┊933┊color-name@^1.0.0:
+┊   ┊934┊  version "1.1.4"
+┊   ┊935┊  resolved "https://registry.yarnpkg.com/color-name/-/color-name-1.1.4.tgz#c2a09a87acbde69543de6f63fa3995c826c536a2"
+┊   ┊936┊  integrity sha512-dOy+3AuW3a2wNbZHIuMZpTcgjGuLU/uBL/ubcZF9OXbDo8ff4O8yVp5Bf0efS8uEoYo5q4Fx7dY9OgQGXgAsQA==
+┊   ┊937┊
+┊   ┊938┊color-string@^1.5.2:
+┊   ┊939┊  version "1.5.3"
+┊   ┊940┊  resolved "https://registry.yarnpkg.com/color-string/-/color-string-1.5.3.tgz#c9bbc5f01b58b5492f3d6857459cb6590ce204cc"
+┊   ┊941┊  integrity sha512-dC2C5qeWoYkxki5UAXapdjqO672AM4vZuPGRQfO8b5HKuKGBbKWpITyDYN7TOFKvRW7kOgAn3746clDBMDJyQw==
+┊   ┊942┊  dependencies:
+┊   ┊943┊    color-name "^1.0.0"
+┊   ┊944┊    simple-swizzle "^0.2.2"
+┊   ┊945┊
+┊   ┊946┊color@3.0.x:
+┊   ┊947┊  version "3.0.0"
+┊   ┊948┊  resolved "https://registry.yarnpkg.com/color/-/color-3.0.0.tgz#d920b4328d534a3ac8295d68f7bd4ba6c427be9a"
+┊   ┊949┊  integrity sha512-jCpd5+s0s0t7p3pHQKpnJ0TpQKKdleP71LWcA0aqiljpiuAkOSUFN/dyH8ZwF0hRmFlrIuRhufds1QyEP9EB+w==
+┊   ┊950┊  dependencies:
+┊   ┊951┊    color-convert "^1.9.1"
+┊   ┊952┊    color-string "^1.5.2"
+┊   ┊953┊
+┊   ┊954┊colornames@^1.1.1:
+┊   ┊955┊  version "1.1.1"
+┊   ┊956┊  resolved "https://registry.yarnpkg.com/colornames/-/colornames-1.1.1.tgz#f8889030685c7c4ff9e2a559f5077eb76a816f96"
+┊   ┊957┊  integrity sha1-+IiQMGhcfE/54qVZ9Qd+t2qBb5Y=
+┊   ┊958┊
+┊   ┊959┊colors@^1.2.1:
+┊   ┊960┊  version "1.3.3"
+┊   ┊961┊  resolved "https://registry.yarnpkg.com/colors/-/colors-1.3.3.tgz#39e005d546afe01e01f9c4ca8fa50f686a01205d"
+┊   ┊962┊  integrity sha512-mmGt/1pZqYRjMxB1axhTo16/snVZ5krrKkcmMeVKxzECMMXoCgnvTPp10QgHfcbQZw8Dq2jMNG6je4JlWU0gWg==
+┊   ┊963┊
+┊   ┊964┊colorspace@1.1.x:
+┊   ┊965┊  version "1.1.1"
+┊   ┊966┊  resolved "https://registry.yarnpkg.com/colorspace/-/colorspace-1.1.1.tgz#9ac2491e1bc6f8fb690e2176814f8d091636d972"
+┊   ┊967┊  integrity sha512-pI3btWyiuz7Ken0BWh9Elzsmv2bM9AhA7psXib4anUXy/orfZ/E0MbQwhSOG/9L8hLlalqrU0UhOuqxW1YjmVw==
+┊   ┊968┊  dependencies:
+┊   ┊969┊    color "3.0.x"
+┊   ┊970┊    text-hex "1.0.x"
+┊   ┊971┊
+┊   ┊972┊combined-stream@^1.0.6, combined-stream@~1.0.6:
+┊   ┊973┊  version "1.0.7"
+┊   ┊974┊  resolved "https://registry.yarnpkg.com/combined-stream/-/combined-stream-1.0.7.tgz#2d1d24317afb8abe95d6d2c0b07b57813539d828"
+┊   ┊975┊  integrity sha512-brWl9y6vOB1xYPZcpZde3N9zDByXTosAeMDo4p1wzo6UMOX4vumB+TP1RZ76sfE6Md68Q0NJSrE/gbezd4Ul+w==
+┊   ┊976┊  dependencies:
+┊   ┊977┊    delayed-stream "~1.0.0"
+┊   ┊978┊
+┊   ┊979┊commander@2.19.0:
+┊   ┊980┊  version "2.19.0"
+┊   ┊981┊  resolved "https://registry.yarnpkg.com/commander/-/commander-2.19.0.tgz#f6198aa84e5b83c46054b94ddedbfed5ee9ff12a"
+┊   ┊982┊  integrity sha512-6tvAOO+D6OENvRAh524Dh9jcfKTYDQAqvqezbCW82xj5X0pSrcpxtvRKHLG0yBY6SD7PSDrJaj+0AiOcKVd1Xg==
+┊   ┊983┊
+┊   ┊984┊common-tags@1.8.0:
+┊   ┊985┊  version "1.8.0"
+┊   ┊986┊  resolved "https://registry.yarnpkg.com/common-tags/-/common-tags-1.8.0.tgz#8e3153e542d4a39e9b10554434afaaf98956a937"
+┊   ┊987┊  integrity sha512-6P6g0uetGpW/sdyUy/iQQCbFF0kWVMSIVSyYz7Zgjcgh8mgw8PQzDNZeyZ5DQ2gM7LBoZPHmnjz8rUthkBG5tw==
+┊   ┊988┊
 ┊632┊989┊component-emitter@^1.2.1:
 ┊633┊990┊  version "1.2.1"
 ┊634┊991┊  resolved "https://registry.yarnpkg.com/component-emitter/-/component-emitter-1.2.1.tgz#137918d6d78283f7df7a6b7c5a63e140e69425e6"
```
```diff
@@ -639,6 +996,21 @@
 ┊ 639┊ 996┊  resolved "https://registry.yarnpkg.com/concat-map/-/concat-map-0.0.1.tgz#d8a96bd77fd68df7793a73036a3ba0d5405d477b"
 ┊ 640┊ 997┊  integrity sha1-2Klr13/Wjfd5OnMDajug1UBdR3s=
 ┊ 641┊ 998┊
+┊    ┊ 999┊concurrently@^4.1.0:
+┊    ┊1000┊  version "4.1.0"
+┊    ┊1001┊  resolved "https://registry.yarnpkg.com/concurrently/-/concurrently-4.1.0.tgz#17fdf067da71210685d9ea554423ef239da30d33"
+┊    ┊1002┊  integrity sha512-pwzXCE7qtOB346LyO9eFWpkFJVO3JQZ/qU/feGeaAHiX1M3Rw3zgXKc5cZ8vSH5DGygkjzLFDzA/pwoQDkRNGg==
+┊    ┊1003┊  dependencies:
+┊    ┊1004┊    chalk "^2.4.1"
+┊    ┊1005┊    date-fns "^1.23.0"
+┊    ┊1006┊    lodash "^4.17.10"
+┊    ┊1007┊    read-pkg "^4.0.1"
+┊    ┊1008┊    rxjs "^6.3.3"
+┊    ┊1009┊    spawn-command "^0.0.2-1"
+┊    ┊1010┊    supports-color "^4.5.0"
+┊    ┊1011┊    tree-kill "^1.1.0"
+┊    ┊1012┊    yargs "^12.0.1"
+┊    ┊1013┊
 ┊ 642┊1014┊configstore@^3.0.0:
 ┊ 643┊1015┊  version "3.1.2"
 ┊ 644┊1016┊  resolved "https://registry.yarnpkg.com/configstore/-/configstore-3.1.2.tgz#c6f25defaeef26df12dd33414b001fe81a543f8f"
```
```diff
@@ -656,6 +1028,14 @@
 ┊ 656┊1028┊  resolved "https://registry.yarnpkg.com/console-control-strings/-/console-control-strings-1.1.0.tgz#3d7cf4464db6446ea644bf4b39507f9851008e8e"
 ┊ 657┊1029┊  integrity sha1-PXz0Rk22RG6mRL9LOVB/mFEAjo4=
 ┊ 658┊1030┊
+┊    ┊1031┊constant-case@^2.0.0:
+┊    ┊1032┊  version "2.0.0"
+┊    ┊1033┊  resolved "https://registry.yarnpkg.com/constant-case/-/constant-case-2.0.0.tgz#4175764d389d3fa9c8ecd29186ed6005243b6a46"
+┊    ┊1034┊  integrity sha1-QXV2TTidP6nI7NKRhu1gBSQ7akY=
+┊    ┊1035┊  dependencies:
+┊    ┊1036┊    snake-case "^2.1.0"
+┊    ┊1037┊    upper-case "^1.1.1"
+┊    ┊1038┊
 ┊ 659┊1039┊content-disposition@0.5.2:
 ┊ 660┊1040┊  version "0.5.2"
 ┊ 661┊1041┊  resolved "https://registry.yarnpkg.com/content-disposition/-/content-disposition-0.5.2.tgz#0cf68bb9ddf5f2be7961c3a85178cb85dba78cb4"
```
```diff
@@ -686,7 +1066,7 @@
 ┊ 686┊1066┊  resolved "https://registry.yarnpkg.com/core-js/-/core-js-3.0.0-beta.13.tgz#7732c69be5e4758887917235fe7c0352c4cb42a1"
 ┊ 687┊1067┊  integrity sha512-16Q43c/3LT9NyePUJKL8nRIQgYWjcBhjJSMWg96PVSxoS0PeE0NHitPI3opBrs9MGGHjte1KoEVr9W63YKlTXQ==
 ┊ 688┊1068┊
-┊ 689┊    ┊core-util-is@~1.0.0:
+┊    ┊1069┊core-util-is@1.0.2, core-util-is@~1.0.0:
 ┊ 690┊1070┊  version "1.0.2"
 ┊ 691┊1071┊  resolved "https://registry.yarnpkg.com/core-util-is/-/core-util-is-1.0.2.tgz#b5fd54220aa2bc5ab57aab7140c940754503c1a7"
 ┊ 692┊1072┊  integrity sha1-tf1UIgqivFq1eqtxQMlAdUUDwac=
```
```diff
@@ -706,6 +1086,14 @@
 ┊ 706┊1086┊  dependencies:
 ┊ 707┊1087┊    capture-stack-trace "^1.0.0"
 ┊ 708┊1088┊
+┊    ┊1089┊cross-fetch@2.2.2:
+┊    ┊1090┊  version "2.2.2"
+┊    ┊1091┊  resolved "https://registry.yarnpkg.com/cross-fetch/-/cross-fetch-2.2.2.tgz#a47ff4f7fc712daba8f6a695a11c948440d45723"
+┊    ┊1092┊  integrity sha1-pH/09/xxLauo9qaVoRyUhEDUVyM=
+┊    ┊1093┊  dependencies:
+┊    ┊1094┊    node-fetch "2.1.2"
+┊    ┊1095┊    whatwg-fetch "2.0.4"
+┊    ┊1096┊
 ┊ 709┊1097┊cross-spawn@^5.0.1:
 ┊ 710┊1098┊  version "5.1.0"
 ┊ 711┊1099┊  resolved "https://registry.yarnpkg.com/cross-spawn/-/cross-spawn-5.1.0.tgz#e8bd0efee58fcff6f8f94510a0a554bbfa235449"
```
```diff
@@ -715,11 +1103,34 @@
 ┊ 715┊1103┊    shebang-command "^1.2.0"
 ┊ 716┊1104┊    which "^1.2.9"
 ┊ 717┊1105┊
+┊    ┊1106┊cross-spawn@^6.0.0:
+┊    ┊1107┊  version "6.0.5"
+┊    ┊1108┊  resolved "https://registry.yarnpkg.com/cross-spawn/-/cross-spawn-6.0.5.tgz#4a5ec7c64dfae22c3a14124dbacdee846d80cbc4"
+┊    ┊1109┊  integrity sha512-eTVLrBSt7fjbDygz805pMnstIs2VTBNkRm0qxZd+M7A5XDdxVRWO5MxGBXZhjY4cqLYLdtrGqRf8mBPmzwSpWQ==
+┊    ┊1110┊  dependencies:
+┊    ┊1111┊    nice-try "^1.0.4"
+┊    ┊1112┊    path-key "^2.0.1"
+┊    ┊1113┊    semver "^5.5.0"
+┊    ┊1114┊    shebang-command "^1.2.0"
+┊    ┊1115┊    which "^1.2.9"
+┊    ┊1116┊
 ┊ 718┊1117┊crypto-random-string@^1.0.0:
 ┊ 719┊1118┊  version "1.0.0"
 ┊ 720┊1119┊  resolved "https://registry.yarnpkg.com/crypto-random-string/-/crypto-random-string-1.0.0.tgz#a230f64f568310e1498009940790ec99545bca7e"
 ┊ 721┊1120┊  integrity sha1-ojD2T1aDEOFJgAmUB5DsmVRbyn4=
 ┊ 722┊1121┊
+┊    ┊1122┊dashdash@^1.12.0:
+┊    ┊1123┊  version "1.14.1"
+┊    ┊1124┊  resolved "https://registry.yarnpkg.com/dashdash/-/dashdash-1.14.1.tgz#853cfa0f7cbe2fed5de20326b8dd581035f6e2f0"
+┊    ┊1125┊  integrity sha1-hTz6D3y+L+1d4gMmuN1YEDX24vA=
+┊    ┊1126┊  dependencies:
+┊    ┊1127┊    assert-plus "^1.0.0"
+┊    ┊1128┊
+┊    ┊1129┊date-fns@^1.23.0, date-fns@^1.27.2:
+┊    ┊1130┊  version "1.30.1"
+┊    ┊1131┊  resolved "https://registry.yarnpkg.com/date-fns/-/date-fns-1.30.1.tgz#2e71bf0b119153dbb4cc4e88d9ea5acfb50dc05c"
+┊    ┊1132┊  integrity sha512-hBSVCvSmWC+QypYObzwGOd9wqdDpOt+0wl0KbU+R+uuZBS1jN8VsD1ss3irQDknRj5NvxiTF6oj/nDRnN/UQNw==
+┊    ┊1133┊
 ┊ 723┊1134┊debug@2.6.9, debug@^2.1.2, debug@^2.2.0, debug@^2.3.3:
 ┊ 724┊1135┊  version "2.6.9"
 ┊ 725┊1136┊  resolved "https://registry.yarnpkg.com/debug/-/debug-2.6.9.tgz#5d128515df134ff327e90a4c93f4e077a536341f"
```
```diff
@@ -734,6 +1145,18 @@
 ┊ 734┊1145┊  dependencies:
 ┊ 735┊1146┊    ms "^2.1.1"
 ┊ 736┊1147┊
+┊    ┊1148┊debug@^4.1.0:
+┊    ┊1149┊  version "4.1.1"
+┊    ┊1150┊  resolved "https://registry.yarnpkg.com/debug/-/debug-4.1.1.tgz#3b72260255109c6b589cee050f1d516139664791"
+┊    ┊1151┊  integrity sha512-pYAIzeRo8J6KPEaJ0VWOh5Pzkbw/RetuzehGM7QRRX5he4fPHx2rdKMB256ehJCkX+XRQm16eZLqLNS8RSZXZw==
+┊    ┊1152┊  dependencies:
+┊    ┊1153┊    ms "^2.1.1"
+┊    ┊1154┊
+┊    ┊1155┊decamelize@^1.2.0:
+┊    ┊1156┊  version "1.2.0"
+┊    ┊1157┊  resolved "https://registry.yarnpkg.com/decamelize/-/decamelize-1.2.0.tgz#f6534d15148269b20352e7bee26f501f9a191290"
+┊    ┊1158┊  integrity sha1-9lNNFRSCabIDUue+4m9QH5oZEpA=
+┊    ┊1159┊
 ┊ 737┊1160┊decode-uri-component@^0.2.0:
 ┊ 738┊1161┊  version "0.2.0"
 ┊ 739┊1162┊  resolved "https://registry.yarnpkg.com/decode-uri-component/-/decode-uri-component-0.2.0.tgz#eb3913333458775cb84cd1a1fae062106bb87545"
```
```diff
@@ -744,6 +1167,11 @@
 ┊ 744┊1167┊  resolved "https://registry.yarnpkg.com/deep-extend/-/deep-extend-0.6.0.tgz#c4fa7c95404a17a9c3e8ca7e1537312b736330ac"
 ┊ 745┊1168┊  integrity sha512-LOHxIOaPYdHlJRtCQfDIVZtfw/ufM8+rVj649RIHzcm/vGwQRXFt6OPqIFWsm2XEMrNIEtWR64sY1LEKD2vAOA==
 ┊ 746┊1169┊
+┊    ┊1170┊deepmerge@3.1.0:
+┊    ┊1171┊  version "3.1.0"
+┊    ┊1172┊  resolved "https://registry.yarnpkg.com/deepmerge/-/deepmerge-3.1.0.tgz#a612626ce4803da410d77554bfd80361599c034d"
+┊    ┊1173┊  integrity sha512-/TnecbwXEdycfbsM2++O3eGiatEFHjjNciHEwJclM+T5Kd94qD1AP+2elP/Mq0L5b9VZJao5znR01Mz6eX8Seg==
+┊    ┊1174┊
 ┊ 747┊1175┊define-properties@^1.1.2:
 ┊ 748┊1176┊  version "1.1.3"
 ┊ 749┊1177┊  resolved "https://registry.yarnpkg.com/define-properties/-/define-properties-1.1.3.tgz#cf88da6cbee26fe6db7094f61d870cbd84cee9f1"
```
```diff
@@ -773,6 +1201,11 @@
 ┊ 773┊1201┊    is-descriptor "^1.0.2"
 ┊ 774┊1202┊    isobject "^3.0.1"
 ┊ 775┊1203┊
+┊    ┊1204┊delayed-stream@~1.0.0:
+┊    ┊1205┊  version "1.0.0"
+┊    ┊1206┊  resolved "https://registry.yarnpkg.com/delayed-stream/-/delayed-stream-1.0.0.tgz#df3ae199acadfb7d440aaae0b29e2272b24ec619"
+┊    ┊1207┊  integrity sha1-3zrhmayt+31ECqrgsp4icrJOxhk=
+┊    ┊1208┊
 ┊ 776┊1209┊delegates@^1.0.0:
 ┊ 777┊1210┊  version "1.0.0"
 ┊ 778┊1211┊  resolved "https://registry.yarnpkg.com/delegates/-/delegates-1.0.0.tgz#84c6e159b81904fdca59a0ef44cd870d31250f9a"
```
```diff
@@ -793,11 +1226,25 @@
 ┊ 793┊1226┊  resolved "https://registry.yarnpkg.com/destroy/-/destroy-1.0.4.tgz#978857442c44749e4206613e37946205826abd80"
 ┊ 794┊1227┊  integrity sha1-l4hXRCxEdJ5CBmE+N5RiBYJqvYA=
 ┊ 795┊1228┊
+┊    ┊1229┊detect-indent@5.0.0:
+┊    ┊1230┊  version "5.0.0"
+┊    ┊1231┊  resolved "https://registry.yarnpkg.com/detect-indent/-/detect-indent-5.0.0.tgz#3871cc0a6a002e8c3e5b3cf7f336264675f06b9d"
+┊    ┊1232┊  integrity sha1-OHHMCmoALow+Wzz38zYmRnXwa50=
+┊    ┊1233┊
 ┊ 796┊1234┊detect-libc@^1.0.2:
 ┊ 797┊1235┊  version "1.0.3"
 ┊ 798┊1236┊  resolved "https://registry.yarnpkg.com/detect-libc/-/detect-libc-1.0.3.tgz#fa137c4bd698edf55cd5cd02ac559f91a4c4ba9b"
 ┊ 799┊1237┊  integrity sha1-+hN8S9aY7fVc1c0CrFWfkaTEups=
 ┊ 800┊1238┊
+┊    ┊1239┊diagnostics@^1.1.1:
+┊    ┊1240┊  version "1.1.1"
+┊    ┊1241┊  resolved "https://registry.yarnpkg.com/diagnostics/-/diagnostics-1.1.1.tgz#cab6ac33df70c9d9a727490ae43ac995a769b22a"
+┊    ┊1242┊  integrity sha512-8wn1PmdunLJ9Tqbx+Fx/ZEuHfJf4NKSN2ZBj7SJC/OWRWha843+WsTjqMe1B5E3p28jqBlp+mJ2fPVxPyNgYKQ==
+┊    ┊1243┊  dependencies:
+┊    ┊1244┊    colorspace "1.1.x"
+┊    ┊1245┊    enabled "1.0.x"
+┊    ┊1246┊    kuler "1.0.x"
+┊    ┊1247┊
 ┊ 801┊1248┊dicer@0.3.0:
 ┊ 802┊1249┊  version "0.3.0"
 ┊ 803┊1250┊  resolved "https://registry.yarnpkg.com/dicer/-/dicer-0.3.0.tgz#eacd98b3bfbf92e8ab5c2fdb71aaac44bb06b872"
```
```diff
@@ -810,6 +1257,13 @@
 ┊ 810┊1257┊  resolved "https://registry.yarnpkg.com/diff/-/diff-3.5.0.tgz#800c0dd1e0a8bfbc95835c202ad220fe317e5a12"
 ┊ 811┊1258┊  integrity sha512-A46qtFgd+g7pDZinpnwiRJtxbC1hpgf0uzP3iG89scHk0AUC7A1TGxf5OiiOUv/JMZR8GOt8hL900hV0bOy5xA==
 ┊ 812┊1259┊
+┊    ┊1260┊dot-case@^2.1.0:
+┊    ┊1261┊  version "2.1.1"
+┊    ┊1262┊  resolved "https://registry.yarnpkg.com/dot-case/-/dot-case-2.1.1.tgz#34dcf37f50a8e93c2b3bca8bb7fb9155c7da3bee"
+┊    ┊1263┊  integrity sha1-NNzzf1Co6TwrO8qLt/uRVcfaO+4=
+┊    ┊1264┊  dependencies:
+┊    ┊1265┊    no-case "^2.2.0"
+┊    ┊1266┊
 ┊ 813┊1267┊dot-prop@^4.1.0:
 ┊ 814┊1268┊  version "4.2.0"
 ┊ 815┊1269┊  resolved "https://registry.yarnpkg.com/dot-prop/-/dot-prop-4.2.0.tgz#1f19e0c2e1aa0e32797c49799f2837ac6af69c57"
```
```diff
@@ -822,16 +1276,55 @@
 ┊ 822┊1276┊  resolved "https://registry.yarnpkg.com/duplexer3/-/duplexer3-0.1.4.tgz#ee01dd1cac0ed3cbc7fdbea37dc0a8f1ce002ce2"
 ┊ 823┊1277┊  integrity sha1-7gHdHKwO08vH/b6jfcCo8c4ALOI=
 ┊ 824┊1278┊
+┊    ┊1279┊ecc-jsbn@~0.1.1:
+┊    ┊1280┊  version "0.1.2"
+┊    ┊1281┊  resolved "https://registry.yarnpkg.com/ecc-jsbn/-/ecc-jsbn-0.1.2.tgz#3a83a904e54353287874c564b7549386849a98c9"
+┊    ┊1282┊  integrity sha1-OoOpBOVDUyh4dMVkt1SThoSamMk=
+┊    ┊1283┊  dependencies:
+┊    ┊1284┊    jsbn "~0.1.0"
+┊    ┊1285┊    safer-buffer "^2.1.0"
+┊    ┊1286┊
 ┊ 825┊1287┊ee-first@1.1.1:
 ┊ 826┊1288┊  version "1.1.1"
 ┊ 827┊1289┊  resolved "https://registry.yarnpkg.com/ee-first/-/ee-first-1.1.1.tgz#590c61156b0ae2f4f0255732a158b266bc56b21d"
 ┊ 828┊1290┊  integrity sha1-WQxhFWsK4vTwJVcyoViyZrxWsh0=
 ┊ 829┊1291┊
+┊    ┊1292┊elegant-spinner@^1.0.1:
+┊    ┊1293┊  version "1.0.1"
+┊    ┊1294┊  resolved "https://registry.yarnpkg.com/elegant-spinner/-/elegant-spinner-1.0.1.tgz#db043521c95d7e303fd8f345bedc3349cfb0729e"
+┊    ┊1295┊  integrity sha1-2wQ1IcldfjA/2PNFvtwzSc+wcp4=
+┊    ┊1296┊
+┊    ┊1297┊enabled@1.0.x:
+┊    ┊1298┊  version "1.0.2"
+┊    ┊1299┊  resolved "https://registry.yarnpkg.com/enabled/-/enabled-1.0.2.tgz#965f6513d2c2d1c5f4652b64a2e3396467fc2f93"
+┊    ┊1300┊  integrity sha1-ll9lE9LC0cX0ZStkouM5ZGf8L5M=
+┊    ┊1301┊  dependencies:
+┊    ┊1302┊    env-variable "0.0.x"
+┊    ┊1303┊
 ┊ 830┊1304┊encodeurl@~1.0.2:
 ┊ 831┊1305┊  version "1.0.2"
 ┊ 832┊1306┊  resolved "https://registry.yarnpkg.com/encodeurl/-/encodeurl-1.0.2.tgz#ad3ff4c86ec2d029322f5a02c3a9a606c95b3f59"
 ┊ 833┊1307┊  integrity sha1-rT/0yG7C0CkyL1oCw6mmBslbP1k=
 ┊ 834┊1308┊
+┊    ┊1309┊end-of-stream@^1.1.0:
+┊    ┊1310┊  version "1.4.1"
+┊    ┊1311┊  resolved "https://registry.yarnpkg.com/end-of-stream/-/end-of-stream-1.4.1.tgz#ed29634d19baba463b6ce6b80a37213eab71ec43"
+┊    ┊1312┊  integrity sha512-1MkrZNvWTKCaigbn+W15elq2BB/L22nqrSY5DKlo3X6+vclJm8Bb5djXJBmEX6fS3+zCh/F4VBK5Z2KxJt4s2Q==
+┊    ┊1313┊  dependencies:
+┊    ┊1314┊    once "^1.4.0"
+┊    ┊1315┊
+┊    ┊1316┊env-variable@0.0.x:
+┊    ┊1317┊  version "0.0.5"
+┊    ┊1318┊  resolved "https://registry.yarnpkg.com/env-variable/-/env-variable-0.0.5.tgz#913dd830bef11e96a039c038d4130604eba37f88"
+┊    ┊1319┊  integrity sha512-zoB603vQReOFvTg5xMl9I1P2PnHsHQQKTEowsKKD7nseUfJq6UWzK+4YtlWUO1nhiQUxe6XMkk+JleSZD1NZFA==
+┊    ┊1320┊
+┊    ┊1321┊error-ex@^1.3.1:
+┊    ┊1322┊  version "1.3.2"
+┊    ┊1323┊  resolved "https://registry.yarnpkg.com/error-ex/-/error-ex-1.3.2.tgz#b4ac40648107fdcdcfae242f428bea8a14d4f1bf"
+┊    ┊1324┊  integrity sha512-7dFHNmqeFSEt2ZBsCriorKnn3Z2pj+fd9kmI6QoWw4//DL+icEBfc0U7qJCisqrTsKTjw4fNFy2pW9OqStD84g==
+┊    ┊1325┊  dependencies:
+┊    ┊1326┊    is-arrayish "^0.2.1"
+┊    ┊1327┊
 ┊ 835┊1328┊es-abstract@^1.5.1:
 ┊ 836┊1329┊  version "1.13.0"
 ┊ 837┊1330┊  resolved "https://registry.yarnpkg.com/es-abstract/-/es-abstract-1.13.0.tgz#ac86145fdd5099d8dd49558ccba2eaf9b88e24e9"
```
```diff
@@ -858,11 +1351,21 @@
 ┊ 858┊1351┊  resolved "https://registry.yarnpkg.com/escape-html/-/escape-html-1.0.3.tgz#0258eae4d3d0c0974de1c169188ef0051d1d1988"
 ┊ 859┊1352┊  integrity sha1-Aljq5NPQwJdN4cFpGI7wBR0dGYg=
 ┊ 860┊1353┊
-┊ 861┊    ┊escape-string-regexp@^1.0.5:
+┊    ┊1354┊escape-string-regexp@^1.0.2, escape-string-regexp@^1.0.5:
 ┊ 862┊1355┊  version "1.0.5"
 ┊ 863┊1356┊  resolved "https://registry.yarnpkg.com/escape-string-regexp/-/escape-string-regexp-1.0.5.tgz#1b61c0562190a8dff6ae3bb2cf0200ca130b86d4"
 ┊ 864┊1357┊  integrity sha1-G2HAViGQqN/2rjuyzwIAyhMLhtQ=
 ┊ 865┊1358┊
+┊    ┊1359┊esprima@^4.0.0:
+┊    ┊1360┊  version "4.0.1"
+┊    ┊1361┊  resolved "https://registry.yarnpkg.com/esprima/-/esprima-4.0.1.tgz#13b04cdb3e6c5d19df91ab6987a8695619b0aa71"
+┊    ┊1362┊  integrity sha512-eGuFFw7Upda+g4p+QHvnW0RyTX/SVeJBDM/gCtMARO0cLuT2HcEKnTPvhjV6aGeqrCB/sbNop0Kszm0jsaWU4A==
+┊    ┊1363┊
+┊    ┊1364┊esutils@^2.0.2:
+┊    ┊1365┊  version "2.0.2"
+┊    ┊1366┊  resolved "https://registry.yarnpkg.com/esutils/-/esutils-2.0.2.tgz#0abf4f1caa5bcb1f7a9d8acc6dea4faaa04bac9b"
+┊    ┊1367┊  integrity sha1-Cr9PHKpbyx96nYrMbepPqqBLrJs=
+┊    ┊1368┊
 ┊ 866┊1369┊etag@~1.8.1:
 ┊ 867┊1370┊  version "1.8.1"
 ┊ 868┊1371┊  resolved "https://registry.yarnpkg.com/etag/-/etag-1.8.1.tgz#41ae2eeb65efa62268aebfea83ac7d79299b0887"
```
```diff
@@ -886,6 +1389,19 @@
 ┊ 886┊1389┊    signal-exit "^3.0.0"
 ┊ 887┊1390┊    strip-eof "^1.0.0"
 ┊ 888┊1391┊
+┊    ┊1392┊execa@^1.0.0:
+┊    ┊1393┊  version "1.0.0"
+┊    ┊1394┊  resolved "https://registry.yarnpkg.com/execa/-/execa-1.0.0.tgz#c6236a5bb4df6d6f15e88e7f017798216749ddd8"
+┊    ┊1395┊  integrity sha512-adbxcyWV46qiHyvSp50TKt05tB4tK3HcmF7/nxfAdhnox83seTDbwnaqKO4sXRy7roHAIFqJP/Rw/AuEbX61LA==
+┊    ┊1396┊  dependencies:
+┊    ┊1397┊    cross-spawn "^6.0.0"
+┊    ┊1398┊    get-stream "^4.0.0"
+┊    ┊1399┊    is-stream "^1.1.0"
+┊    ┊1400┊    npm-run-path "^2.0.0"
+┊    ┊1401┊    p-finally "^1.0.0"
+┊    ┊1402┊    signal-exit "^3.0.0"
+┊    ┊1403┊    strip-eof "^1.0.0"
+┊    ┊1404┊
 ┊ 889┊1405┊expand-brackets@^2.1.4:
 ┊ 890┊1406┊  version "2.1.4"
 ┊ 891┊1407┊  resolved "https://registry.yarnpkg.com/expand-brackets/-/expand-brackets-2.1.4.tgz#b77735e315ce30f6b6eff0f83b04151a22449622"
```
```diff
@@ -950,6 +1466,20 @@
 ┊ 950┊1466┊    assign-symbols "^1.0.0"
 ┊ 951┊1467┊    is-extendable "^1.0.1"
 ┊ 952┊1468┊
+┊    ┊1469┊extend@~3.0.2:
+┊    ┊1470┊  version "3.0.2"
+┊    ┊1471┊  resolved "https://registry.yarnpkg.com/extend/-/extend-3.0.2.tgz#f8b1136b4071fbd8eb140aff858b1019ec2915fa"
+┊    ┊1472┊  integrity sha512-fjquC59cD7CyW6urNXK0FBufkZcoiGG80wTuPujX590cB5Ttln20E2UB4S/WARVqhXffZl2LNgS+gQdPIIim/g==
+┊    ┊1473┊
+┊    ┊1474┊external-editor@^3.0.3:
+┊    ┊1475┊  version "3.0.3"
+┊    ┊1476┊  resolved "https://registry.yarnpkg.com/external-editor/-/external-editor-3.0.3.tgz#5866db29a97826dbe4bf3afd24070ead9ea43a27"
+┊    ┊1477┊  integrity sha512-bn71H9+qWoOQKyZDo25mOMVpSmXROAsTJVVVYzrrtol3d4y+AsKjf4Iwl2Q+IuT0kFSQ1qo166UuIwqYq7mGnA==
+┊    ┊1478┊  dependencies:
+┊    ┊1479┊    chardet "^0.7.0"
+┊    ┊1480┊    iconv-lite "^0.4.24"
+┊    ┊1481┊    tmp "^0.0.33"
+┊    ┊1482┊
 ┊ 953┊1483┊extglob@^2.0.4:
 ┊ 954┊1484┊  version "2.0.4"
 ┊ 955┊1485┊  resolved "https://registry.yarnpkg.com/extglob/-/extglob-2.0.4.tgz#ad00fe4dc612a9232e8718711dc5cb5ab0285543"
```
```diff
@@ -964,11 +1494,51 @@
 ┊ 964┊1494┊    snapdragon "^0.8.1"
 ┊ 965┊1495┊    to-regex "^3.0.1"
 ┊ 966┊1496┊
+┊    ┊1497┊extsprintf@1.3.0:
+┊    ┊1498┊  version "1.3.0"
+┊    ┊1499┊  resolved "https://registry.yarnpkg.com/extsprintf/-/extsprintf-1.3.0.tgz#96918440e3041a7a414f8c52e3c574eb3c3e1e05"
+┊    ┊1500┊  integrity sha1-lpGEQOMEGnpBT4xS48V06zw+HgU=
+┊    ┊1501┊
+┊    ┊1502┊extsprintf@^1.2.0:
+┊    ┊1503┊  version "1.4.0"
+┊    ┊1504┊  resolved "https://registry.yarnpkg.com/extsprintf/-/extsprintf-1.4.0.tgz#e2689f8f356fad62cca65a3a91c5df5f9551692f"
+┊    ┊1505┊  integrity sha1-4mifjzVvrWLMplo6kcXfX5VRaS8=
+┊    ┊1506┊
+┊    ┊1507┊fast-deep-equal@^2.0.1:
+┊    ┊1508┊  version "2.0.1"
+┊    ┊1509┊  resolved "https://registry.yarnpkg.com/fast-deep-equal/-/fast-deep-equal-2.0.1.tgz#7b05218ddf9667bf7f370bf7fdb2cb15fdd0aa49"
+┊    ┊1510┊  integrity sha1-ewUhjd+WZ79/Nwv3/bLLFf3Qqkk=
+┊    ┊1511┊
 ┊ 967┊1512┊fast-json-stable-stringify@^2.0.0:
 ┊ 968┊1513┊  version "2.0.0"
 ┊ 969┊1514┊  resolved "https://registry.yarnpkg.com/fast-json-stable-stringify/-/fast-json-stable-stringify-2.0.0.tgz#d5142c0caee6b1189f87d3a76111064f86c8bbf2"
 ┊ 970┊1515┊  integrity sha1-1RQsDK7msRifh9OnYREGT4bIu/I=
 ┊ 971┊1516┊
+┊    ┊1517┊fast-safe-stringify@^2.0.4:
+┊    ┊1518┊  version "2.0.6"
+┊    ┊1519┊  resolved "https://registry.yarnpkg.com/fast-safe-stringify/-/fast-safe-stringify-2.0.6.tgz#04b26106cc56681f51a044cfc0d76cf0008ac2c2"
+┊    ┊1520┊  integrity sha512-q8BZ89jjc+mz08rSxROs8VsrBBcn1SIw1kq9NjolL509tkABRk9io01RAjSaEv1Xb2uFLt8VtRiZbGp5H8iDtg==
+┊    ┊1521┊
+┊    ┊1522┊fecha@^2.3.3:
+┊    ┊1523┊  version "2.3.3"
+┊    ┊1524┊  resolved "https://registry.yarnpkg.com/fecha/-/fecha-2.3.3.tgz#948e74157df1a32fd1b12c3a3c3cdcb6ec9d96cd"
+┊    ┊1525┊  integrity sha512-lUGBnIamTAwk4znq5BcqsDaxSmZ9nDVJaij6NvRt/Tg4R69gERA+otPKbS86ROw9nxVMw2/mp1fnaiWqbs6Sdg==
+┊    ┊1526┊
+┊    ┊1527┊figures@^1.7.0:
+┊    ┊1528┊  version "1.7.0"
+┊    ┊1529┊  resolved "https://registry.yarnpkg.com/figures/-/figures-1.7.0.tgz#cbe1e3affcf1cd44b80cadfed28dc793a9701d2e"
+┊    ┊1530┊  integrity sha1-y+Hjr/zxzUS4DK3+0o3Hk6lwHS4=
+┊    ┊1531┊  dependencies:
+┊    ┊1532┊    escape-string-regexp "^1.0.5"
+┊    ┊1533┊    object-assign "^4.1.0"
+┊    ┊1534┊
+┊    ┊1535┊figures@^2.0.0:
+┊    ┊1536┊  version "2.0.0"
+┊    ┊1537┊  resolved "https://registry.yarnpkg.com/figures/-/figures-2.0.0.tgz#3ab1a2d2a62c8bfb431a0c94cb797a2fce27c962"
+┊    ┊1538┊  integrity sha1-OrGi0qYsi/tDGgyUy3l6L84nyWI=
+┊    ┊1539┊  dependencies:
+┊    ┊1540┊    escape-string-regexp "^1.0.5"
+┊    ┊1541┊
 ┊ 972┊1542┊fill-range@^4.0.0:
 ┊ 973┊1543┊  version "4.0.0"
 ┊ 974┊1544┊  resolved "https://registry.yarnpkg.com/fill-range/-/fill-range-4.0.0.tgz#d544811d428f98eb06a63dc402d2403c328c38f7"
```
```diff
@@ -992,11 +1562,32 @@
 ┊ 992┊1562┊    statuses "~1.4.0"
 ┊ 993┊1563┊    unpipe "~1.0.0"
 ┊ 994┊1564┊
+┊    ┊1565┊find-up@^3.0.0:
+┊    ┊1566┊  version "3.0.0"
+┊    ┊1567┊  resolved "https://registry.yarnpkg.com/find-up/-/find-up-3.0.0.tgz#49169f1d7993430646da61ecc5ae355c21c97b73"
+┊    ┊1568┊  integrity sha512-1yD6RmLI1XBfxugvORwlck6f75tYL+iR0jqwsOrOxMZyGYqUuDhJ0l4AXdO1iX/FTs9cBAMEk1gWSEx1kSbylg==
+┊    ┊1569┊  dependencies:
+┊    ┊1570┊    locate-path "^3.0.0"
+┊    ┊1571┊
 ┊ 995┊1572┊for-in@^1.0.2:
 ┊ 996┊1573┊  version "1.0.2"
 ┊ 997┊1574┊  resolved "https://registry.yarnpkg.com/for-in/-/for-in-1.0.2.tgz#81068d295a8142ec0ac726c6e2200c30fb6d5e80"
 ┊ 998┊1575┊  integrity sha1-gQaNKVqBQuwKxybG4iAMMPttXoA=
 ┊ 999┊1576┊
+┊    ┊1577┊forever-agent@~0.6.1:
+┊    ┊1578┊  version "0.6.1"
+┊    ┊1579┊  resolved "https://registry.yarnpkg.com/forever-agent/-/forever-agent-0.6.1.tgz#fbc71f0c41adeb37f96c577ad1ed42d8fdacca91"
+┊    ┊1580┊  integrity sha1-+8cfDEGt6zf5bFd60e1C2P2sypE=
+┊    ┊1581┊
+┊    ┊1582┊form-data@~2.3.2:
+┊    ┊1583┊  version "2.3.3"
+┊    ┊1584┊  resolved "https://registry.yarnpkg.com/form-data/-/form-data-2.3.3.tgz#dcce52c05f644f298c6a7ab936bd724ceffbf3a6"
+┊    ┊1585┊  integrity sha512-1lLKB2Mu3aGP1Q/2eCOx0fNbRMe7XdwktwOruhfqqd0rIJWwN4Dh+E3hrPSlDCXnSR7UtZ1N38rVXm+6+MEhJQ==
+┊    ┊1586┊  dependencies:
+┊    ┊1587┊    asynckit "^0.4.0"
+┊    ┊1588┊    combined-stream "^1.0.6"
+┊    ┊1589┊    mime-types "^2.1.12"
+┊    ┊1590┊
 ┊1000┊1591┊forwarded@~0.1.2:
 ┊1001┊1592┊  version "0.1.2"
 ┊1002┊1593┊  resolved "https://registry.yarnpkg.com/forwarded/-/forwarded-0.1.2.tgz#98c23dab1175657b8c0573e8ceccd91b0ff18c84"
```
```diff
@@ -1058,16 +1649,35 @@
 ┊1058┊1649┊    strip-ansi "^3.0.1"
 ┊1059┊1650┊    wide-align "^1.1.0"
 ┊1060┊1651┊
+┊    ┊1652┊get-caller-file@^1.0.1:
+┊    ┊1653┊  version "1.0.3"
+┊    ┊1654┊  resolved "https://registry.yarnpkg.com/get-caller-file/-/get-caller-file-1.0.3.tgz#f978fa4c90d1dfe7ff2d6beda2a515e713bdcf4a"
+┊    ┊1655┊  integrity sha512-3t6rVToeoZfYSGd8YoLFR2DJkiQrIiUrGcjvFX2mDw3bn6k2OtwHN0TNCLbBO+w8qTvimhDkv+LSscbJY1vE6w==
+┊    ┊1656┊
 ┊1061┊1657┊get-stream@^3.0.0:
 ┊1062┊1658┊  version "3.0.0"
 ┊1063┊1659┊  resolved "https://registry.yarnpkg.com/get-stream/-/get-stream-3.0.0.tgz#8e943d1358dc37555054ecbe2edb05aa174ede14"
 ┊1064┊1660┊  integrity sha1-jpQ9E1jcN1VQVOy+LtsFqhdO3hQ=
 ┊1065┊1661┊
+┊    ┊1662┊get-stream@^4.0.0:
+┊    ┊1663┊  version "4.1.0"
+┊    ┊1664┊  resolved "https://registry.yarnpkg.com/get-stream/-/get-stream-4.1.0.tgz#c1b255575f3dc21d59bfc79cd3d2b46b1c3a54b5"
+┊    ┊1665┊  integrity sha512-GMat4EJ5161kIy2HevLlr4luNjBgvmj413KaQA7jt4V8B4RDsfpHk7WQ9GVqfYyyx8OS/L66Kox+rJRNklLK7w==
+┊    ┊1666┊  dependencies:
+┊    ┊1667┊    pump "^3.0.0"
+┊    ┊1668┊
 ┊1066┊1669┊get-value@^2.0.3, get-value@^2.0.6:
 ┊1067┊1670┊  version "2.0.6"
 ┊1068┊1671┊  resolved "https://registry.yarnpkg.com/get-value/-/get-value-2.0.6.tgz#dc15ca1c672387ca76bd37ac0a395ba2042a2c28"
 ┊1069┊1672┊  integrity sha1-3BXKHGcjh8p2vTesCjlbogQqLCg=
 ┊1070┊1673┊
+┊    ┊1674┊getpass@^0.1.1:
+┊    ┊1675┊  version "0.1.7"
+┊    ┊1676┊  resolved "https://registry.yarnpkg.com/getpass/-/getpass-0.1.7.tgz#5eff8e3e684d569ae4cb2b1282604e8ba62149fa"
+┊    ┊1677┊  integrity sha1-Xv+OPmhNVprkyysSgmBOi6YhSfo=
+┊    ┊1678┊  dependencies:
+┊    ┊1679┊    assert-plus "^1.0.0"
+┊    ┊1680┊
 ┊1071┊1681┊glob-parent@^3.1.0:
 ┊1072┊1682┊  version "3.1.0"
 ┊1073┊1683┊  resolved "https://registry.yarnpkg.com/glob-parent/-/glob-parent-3.1.0.tgz#9e6af6299d8d3bd2bd40430832bd113df906c5ae"
```
```diff
@@ -1076,7 +1686,7 @@
 ┊1076┊1686┊    is-glob "^3.1.0"
 ┊1077┊1687┊    path-dirname "^1.0.0"
 ┊1078┊1688┊
-┊1079┊    ┊glob@^7.1.3:
+┊    ┊1689┊glob@7.1.3, glob@^7.1.3:
 ┊1080┊1690┊  version "7.1.3"
 ┊1081┊1691┊  resolved "https://registry.yarnpkg.com/glob/-/glob-7.1.3.tgz#3960832d3f1574108342dafd3a67b332c0969df1"
 ┊1082┊1692┊  integrity sha512-vcfuiIxogLV4DlGBHIUOwI0IbrJ8HWPc4MU7HzviGeNho/UJDfi6B5p3sHeWIQ0KGIU0Jpxi5ZHxemQfLkkAwQ==
```
```diff
@@ -1095,6 +1705,11 @@
 ┊1095┊1705┊  dependencies:
 ┊1096┊1706┊    ini "^1.3.4"
 ┊1097┊1707┊
+┊    ┊1708┊globals@^11.1.0:
+┊    ┊1709┊  version "11.11.0"
+┊    ┊1710┊  resolved "https://registry.yarnpkg.com/globals/-/globals-11.11.0.tgz#dcf93757fa2de5486fbeed7118538adf789e9c2e"
+┊    ┊1711┊  integrity sha512-WHq43gS+6ufNOEqlrDBxVEbb8ntfXrfAUU2ZOpCxrBdGKW3gyv8mCxAfIBD0DroPKGrJ2eSsXsLtY9MPntsyTw==
+┊    ┊1712┊
 ┊1098┊1713┊got@^6.7.1:
 ┊1099┊1714┊  version "6.7.1"
 ┊1100┊1715┊  resolved "https://registry.yarnpkg.com/got/-/got-6.7.1.tgz#240cd05785a9a18e561dc1b44b41c763ef1e8db0"
```
```diff
@@ -1117,6 +1732,103 @@
 ┊1117┊1732┊  resolved "https://registry.yarnpkg.com/graceful-fs/-/graceful-fs-4.1.15.tgz#ffb703e1066e8a0eeaa4c8b80ba9253eeefbfb00"
 ┊1118┊1733┊  integrity sha512-6uHUhOPEBgQ24HM+r6b/QwWfZq+yiFcipKFrOFiBEnWdy5sdzYoi+pJeQaPI5qOLRFqWmAXUPQNsielzdLoecA==
 ┊1119┊1734┊
+┊    ┊1735┊graphql-code-generator@^0.17.0:
+┊    ┊1736┊  version "0.17.0"
+┊    ┊1737┊  resolved "https://registry.yarnpkg.com/graphql-code-generator/-/graphql-code-generator-0.17.0.tgz#7e76f7c2e7df69af89c478094137385c2761ce4d"
+┊    ┊1738┊  integrity sha512-91KIkyAhdfoYvd2tArwvPql14iaC9x9PN0CmFM2ubnceQZFsIUgPHr4Gmpv+YNYSY97ePmuP6Ui0IHZydvrTPw==
+┊    ┊1739┊  dependencies:
+┊    ┊1740┊    "@types/babylon" "6.16.5"
+┊    ┊1741┊    "@types/is-glob" "4.0.0"
+┊    ┊1742┊    "@types/prettier" "1.16.1"
+┊    ┊1743┊    "@types/valid-url" "1.0.2"
+┊    ┊1744┊    babel-types "7.0.0-beta.3"
+┊    ┊1745┊    babylon "7.0.0-beta.47"
+┊    ┊1746┊    chalk "2.4.2"
+┊    ┊1747┊    change-case "3.1.0"
+┊    ┊1748┊    chokidar "2.1.2"
+┊    ┊1749┊    commander "2.19.0"
+┊    ┊1750┊    common-tags "1.8.0"
+┊    ┊1751┊    detect-indent "5.0.0"
+┊    ┊1752┊    glob "7.1.3"
+┊    ┊1753┊    graphql-codegen-core "0.17.0"
+┊    ┊1754┊    graphql-config "2.2.1"
+┊    ┊1755┊    graphql-import "0.7.1"
+┊    ┊1756┊    graphql-tag-pluck "0.5.0"
+┊    ┊1757┊    graphql-toolkit "0.0.5"
+┊    ┊1758┊    graphql-tools "4.0.4"
+┊    ┊1759┊    indent-string "3.2.0"
+┊    ┊1760┊    inquirer "6.2.2"
+┊    ┊1761┊    is-glob "4.0.0"
+┊    ┊1762┊    is-valid-path "0.1.1"
+┊    ┊1763┊    js-yaml "3.12.1"
+┊    ┊1764┊    json-to-pretty-yaml "1.2.2"
+┊    ┊1765┊    listr "0.14.3"
+┊    ┊1766┊    listr-update-renderer "0.5.0"
+┊    ┊1767┊    log-symbols "2.2.0"
+┊    ┊1768┊    log-update "2.3.0"
+┊    ┊1769┊    mkdirp "0.5.1"
+┊    ┊1770┊    prettier "1.16.4"
+┊    ┊1771┊    request "2.88.0"
+┊    ┊1772┊    valid-url "1.0.9"
+┊    ┊1773┊
+┊    ┊1774┊graphql-codegen-core@0.17.0:
+┊    ┊1775┊  version "0.17.0"
+┊    ┊1776┊  resolved "https://registry.yarnpkg.com/graphql-codegen-core/-/graphql-codegen-core-0.17.0.tgz#e0e23cf0e1138ff3e75912604cf04aae82aa27be"
+┊    ┊1777┊  integrity sha512-3WULDb+XaRrSsVbTU8VBf7yzGGnW9pAKl+wE3p2NjaKPm0IPBKlWrDtVCUPpTXE0oANv0V4ytsHjMMJ8oTtRLg==
+┊    ┊1778┊  dependencies:
+┊    ┊1779┊    chalk "2.4.2"
+┊    ┊1780┊    change-case "3.1.0"
+┊    ┊1781┊    common-tags "1.8.0"
+┊    ┊1782┊    graphql-tag "2.10.1"
+┊    ┊1783┊    graphql-toolkit "0.0.5"
+┊    ┊1784┊    graphql-tools "4.0.4"
+┊    ┊1785┊    ts-log "2.1.4"
+┊    ┊1786┊    winston "3.2.1"
+┊    ┊1787┊
+┊    ┊1788┊graphql-codegen-plugin-helpers@0.17.0:
+┊    ┊1789┊  version "0.17.0"
+┊    ┊1790┊  resolved "https://registry.yarnpkg.com/graphql-codegen-plugin-helpers/-/graphql-codegen-plugin-helpers-0.17.0.tgz#f65edde8a0810c457a85171d2e8bd0d29c8c7ac3"
+┊    ┊1791┊  integrity sha512-zcTvtvJqGrGTUjM9qz4vGml967uORX5yeSBc9n4du+Ah+y4TLG0GdjmEqfP7Jokwb8YpCGKivY6hXF8AZ7yPdw==
+┊    ┊1792┊  dependencies:
+┊    ┊1793┊    graphql-codegen-core "0.17.0"
+┊    ┊1794┊    import-from "2.1.0"
+┊    ┊1795┊
+┊    ┊1796┊graphql-codegen-typescript-common@0.17.0, graphql-codegen-typescript-common@^0.17.0:
+┊    ┊1797┊  version "0.17.0"
+┊    ┊1798┊  resolved "https://registry.yarnpkg.com/graphql-codegen-typescript-common/-/graphql-codegen-typescript-common-0.17.0.tgz#76cfeefa92ee8420f2c9a687576fd5ac3d6e14fb"
+┊    ┊1799┊  integrity sha512-tUaJB4fuHHgZa1JFroe5V4kIn69HKmKwaHEvizXhuAmJUnHGThzjnn71Ot0FMSpF6RLcMf/adDsNts/BQrUEYg==
+┊    ┊1800┊  dependencies:
+┊    ┊1801┊    change-case "3.1.0"
+┊    ┊1802┊    common-tags "1.8.0"
+┊    ┊1803┊    graphql-codegen-core "0.17.0"
+┊    ┊1804┊    graphql-codegen-plugin-helpers "0.17.0"
+┊    ┊1805┊
+┊    ┊1806┊graphql-codegen-typescript-resolvers@^0.17.0:
+┊    ┊1807┊  version "0.17.0"
+┊    ┊1808┊  resolved "https://registry.yarnpkg.com/graphql-codegen-typescript-resolvers/-/graphql-codegen-typescript-resolvers-0.17.0.tgz#b08ad97c36f742fe08acafc28f19192021873049"
+┊    ┊1809┊  integrity sha512-yFYUZ0mXNIhzBKWv1qWnDWPm9SnXR30AfAFsp2DGY6v5834NS48pLHsiHp3z+Fgv+MvANhW+tI1LeViB0Ds5jg==
+┊    ┊1810┊  dependencies:
+┊    ┊1811┊    graphql-codegen-plugin-helpers "0.17.0"
+┊    ┊1812┊    graphql-codegen-typescript-common "0.17.0"
+┊    ┊1813┊
+┊    ┊1814┊graphql-codegen-typescript-server@^0.17.0:
+┊    ┊1815┊  version "0.17.0"
+┊    ┊1816┊  resolved "https://registry.yarnpkg.com/graphql-codegen-typescript-server/-/graphql-codegen-typescript-server-0.17.0.tgz#6c6e70cfe72d73db33662469d83c3a0edcaf3178"
+┊    ┊1817┊  integrity sha512-70EqTJTJ1K8nVvhkpTdzmY4k04/OV1KsGEZeQeH7rPtVxJ2acrxlpFAJIqjQgsCPSnuZJotQfZDNAWmn4bd5nQ==
+┊    ┊1818┊  dependencies:
+┊    ┊1819┊    graphql-codegen-typescript-common "0.17.0"
+┊    ┊1820┊
+┊    ┊1821┊graphql-config@2.2.1:
+┊    ┊1822┊  version "2.2.1"
+┊    ┊1823┊  resolved "https://registry.yarnpkg.com/graphql-config/-/graphql-config-2.2.1.tgz#5fd0ec77ac7428ca5fb2026cf131be10151a0cb2"
+┊    ┊1824┊  integrity sha512-U8+1IAhw9m6WkZRRcyj8ZarK96R6lQBQ0an4lp76Ps9FyhOXENC5YQOxOFGm5CxPrX2rD0g3Je4zG5xdNJjwzQ==
+┊    ┊1825┊  dependencies:
+┊    ┊1826┊    graphql-import "^0.7.1"
+┊    ┊1827┊    graphql-request "^1.5.0"
+┊    ┊1828┊    js-yaml "^3.10.0"
+┊    ┊1829┊    lodash "^4.17.4"
+┊    ┊1830┊    minimatch "^3.0.4"
+┊    ┊1831┊
 ┊1120┊1832┊graphql-extensions@0.5.4:
 ┊1121┊1833┊  version "0.5.4"
 ┊1122┊1834┊  resolved "https://registry.yarnpkg.com/graphql-extensions/-/graphql-extensions-0.5.4.tgz#18a9674f9adb11aa6c0737485887ea8877914cff"
```
```diff
@@ -1131,11 +1843,26 @@
 ┊1131┊1843┊  dependencies:
 ┊1132┊1844┊    "@apollographql/apollo-tools" "^0.3.3"
 ┊1133┊1845┊
+┊    ┊1846┊graphql-import@0.7.1, graphql-import@^0.7.1:
+┊    ┊1847┊  version "0.7.1"
+┊    ┊1848┊  resolved "https://registry.yarnpkg.com/graphql-import/-/graphql-import-0.7.1.tgz#4add8d91a5f752d764b0a4a7a461fcd93136f223"
+┊    ┊1849┊  integrity sha512-YpwpaPjRUVlw2SN3OPljpWbVRWAhMAyfSba5U47qGMOSsPLi2gYeJtngGpymjm9nk57RFWEpjqwh4+dpYuFAPw==
+┊    ┊1850┊  dependencies:
+┊    ┊1851┊    lodash "^4.17.4"
+┊    ┊1852┊    resolve-from "^4.0.0"
+┊    ┊1853┊
 ┊1134┊1854┊graphql-iso-date@^3.6.1:
 ┊1135┊1855┊  version "3.6.1"
 ┊1136┊1856┊  resolved "https://registry.yarnpkg.com/graphql-iso-date/-/graphql-iso-date-3.6.1.tgz#bd2d0dc886e0f954cbbbc496bbf1d480b57ffa96"
 ┊1137┊1857┊  integrity sha512-AwFGIuYMJQXOEAgRlJlFL4H1ncFM8n8XmoVDTNypNOZyQ8LFDG2ppMFlsS862BSTCDcSUfHp8PD3/uJhv7t59Q==
 ┊1138┊1858┊
+┊    ┊1859┊graphql-request@^1.5.0:
+┊    ┊1860┊  version "1.8.2"
+┊    ┊1861┊  resolved "https://registry.yarnpkg.com/graphql-request/-/graphql-request-1.8.2.tgz#398d10ae15c585676741bde3fc01d5ca948f8fbe"
+┊    ┊1862┊  integrity sha512-dDX2M+VMsxXFCmUX0Vo0TopIZIX4ggzOtiCsThgtrKR4niiaagsGTDIHj3fsOMFETpa064vzovI+4YV4QnMbcg==
+┊    ┊1863┊  dependencies:
+┊    ┊1864┊    cross-fetch "2.2.2"
+┊    ┊1865┊
 ┊1139┊1866┊graphql-subscriptions@^1.0.0:
 ┊1140┊1867┊  version "1.0.0"
 ┊1141┊1868┊  resolved "https://registry.yarnpkg.com/graphql-subscriptions/-/graphql-subscriptions-1.0.0.tgz#475267694b3bd465af6477dbab4263a3f62702b8"
```
```diff
@@ -1143,12 +1870,38 @@
 ┊1143┊1870┊  dependencies:
 ┊1144┊1871┊    iterall "^1.2.1"
 ┊1145┊1872┊
-┊1146┊    ┊graphql-tag@^2.9.2:
+┊    ┊1873┊graphql-tag-pluck@0.5.0:
+┊    ┊1874┊  version "0.5.0"
+┊    ┊1875┊  resolved "https://registry.yarnpkg.com/graphql-tag-pluck/-/graphql-tag-pluck-0.5.0.tgz#81f5dee3a6ca829f205ab032336be7b107398b2e"
+┊    ┊1876┊  integrity sha512-SlsIpXKbrKIV2+QxYZ7bFPQ0DpIXFd0BEz3U+Krt8tuHYMFtElcctNDppW4EeQTrSX9H5zpStBZyfOYQjQOH1w==
+┊    ┊1877┊  dependencies:
+┊    ┊1878┊    "@babel/parser" "^7.2.0"
+┊    ┊1879┊    "@babel/traverse" "^7.1.6"
+┊    ┊1880┊    "@babel/types" "^7.2.0"
+┊    ┊1881┊    source-map-support "^0.5.9"
+┊    ┊1882┊    typescript "^3.2.2"
+┊    ┊1883┊
+┊    ┊1884┊graphql-tag@2.10.1, graphql-tag@^2.9.2:
 ┊1147┊1885┊  version "2.10.1"
 ┊1148┊1886┊  resolved "https://registry.yarnpkg.com/graphql-tag/-/graphql-tag-2.10.1.tgz#10aa41f1cd8fae5373eaf11f1f67260a3cad5e02"
 ┊1149┊1887┊  integrity sha512-jApXqWBzNXQ8jYa/HLkZJaVw9jgwNqZkywa2zfFn16Iv1Zb7ELNHkJaXHR7Quvd5SIGsy6Ny7SUKATgnu05uEg==
 ┊1150┊1888┊
-┊1151┊    ┊graphql-tools@^4.0.0:
+┊    ┊1889┊graphql-toolkit@0.0.5:
+┊    ┊1890┊  version "0.0.5"
+┊    ┊1891┊  resolved "https://registry.yarnpkg.com/graphql-toolkit/-/graphql-toolkit-0.0.5.tgz#9e6ebe3d4b33fc329e5ee3b7775bfe7fba2f48a5"
+┊    ┊1892┊  integrity sha512-655RP1y8cn65mOa9EE/jnttczHE0lFXpOV1zYLTsE1A0b5j8RVuKWllSZBnnL2WHSAPPqLZ1oJEZV2uzSdV9VQ==
+┊    ┊1893┊  dependencies:
+┊    ┊1894┊    deepmerge "3.1.0"
+┊    ┊1895┊    glob "7.1.3"
+┊    ┊1896┊    graphql-import "0.7.1"
+┊    ┊1897┊    graphql-tag-pluck "0.5.0"
+┊    ┊1898┊    is-glob "4.0.0"
+┊    ┊1899┊    is-valid-path "0.1.1"
+┊    ┊1900┊    lodash "4.17.11"
+┊    ┊1901┊    request "2.88.0"
+┊    ┊1902┊    valid-url "1.0.9"
+┊    ┊1903┊
+┊    ┊1904┊graphql-tools@4.0.4, graphql-tools@^4.0.0:
 ┊1152┊1905┊  version "4.0.4"
 ┊1153┊1906┊  resolved "https://registry.yarnpkg.com/graphql-tools/-/graphql-tools-4.0.4.tgz#ca08a63454221fdde825fe45fbd315eb2a6d566b"
 ┊1154┊1907┊  integrity sha512-chF12etTIGVVGy3fCTJ1ivJX2KB7OSG4c6UOJQuqOHCmBQwTyNgCDuejZKvpYxNZiEx7bwIjrodDgDe9RIkjlw==
```
```diff
@@ -1176,6 +1929,31 @@
 ┊1176┊1929┊  dependencies:
 ┊1177┊1930┊    iterall "^1.2.2"
 ┊1178┊1931┊
+┊    ┊1932┊har-schema@^2.0.0:
+┊    ┊1933┊  version "2.0.0"
+┊    ┊1934┊  resolved "https://registry.yarnpkg.com/har-schema/-/har-schema-2.0.0.tgz#a94c2224ebcac04782a0d9035521f24735b7ec92"
+┊    ┊1935┊  integrity sha1-qUwiJOvKwEeCoNkDVSHyRzW37JI=
+┊    ┊1936┊
+┊    ┊1937┊har-validator@~5.1.0:
+┊    ┊1938┊  version "5.1.3"
+┊    ┊1939┊  resolved "https://registry.yarnpkg.com/har-validator/-/har-validator-5.1.3.tgz#1ef89ebd3e4996557675eed9893110dc350fa080"
+┊    ┊1940┊  integrity sha512-sNvOCzEQNr/qrvJgc3UG/kD4QtlHycrzwS+6mfTrrSq97BvaYcPZZI1ZSqGSPR73Cxn4LKTD4PttRwfU7jWq5g==
+┊    ┊1941┊  dependencies:
+┊    ┊1942┊    ajv "^6.5.5"
+┊    ┊1943┊    har-schema "^2.0.0"
+┊    ┊1944┊
+┊    ┊1945┊has-ansi@^2.0.0:
+┊    ┊1946┊  version "2.0.0"
+┊    ┊1947┊  resolved "https://registry.yarnpkg.com/has-ansi/-/has-ansi-2.0.0.tgz#34f5049ce1ecdf2b0649af3ef24e45ed35416d91"
+┊    ┊1948┊  integrity sha1-NPUEnOHs3ysGSa8+8k5F7TVBbZE=
+┊    ┊1949┊  dependencies:
+┊    ┊1950┊    ansi-regex "^2.0.0"
+┊    ┊1951┊
+┊    ┊1952┊has-flag@^2.0.0:
+┊    ┊1953┊  version "2.0.0"
+┊    ┊1954┊  resolved "https://registry.yarnpkg.com/has-flag/-/has-flag-2.0.0.tgz#e8207af1cc7b30d446cc70b734b5e8be18f88d51"
+┊    ┊1955┊  integrity sha1-6CB68cx7MNRGzHC3NLXovhj4jVE=
+┊    ┊1956┊
 ┊1179┊1957┊has-flag@^3.0.0:
 ┊1180┊1958┊  version "3.0.0"
 ┊1181┊1959┊  resolved "https://registry.yarnpkg.com/has-flag/-/has-flag-3.0.0.tgz#b5d454dc2199ae225699f3467e5a07f3b955bafd"
```
```diff
@@ -1229,6 +2007,19 @@
 ┊1229┊2007┊  dependencies:
 ┊1230┊2008┊    function-bind "^1.1.1"
 ┊1231┊2009┊
+┊    ┊2010┊header-case@^1.0.0:
+┊    ┊2011┊  version "1.0.1"
+┊    ┊2012┊  resolved "https://registry.yarnpkg.com/header-case/-/header-case-1.0.1.tgz#9535973197c144b09613cd65d317ef19963bd02d"
+┊    ┊2013┊  integrity sha1-lTWXMZfBRLCWE81l0xfvGZY70C0=
+┊    ┊2014┊  dependencies:
+┊    ┊2015┊    no-case "^2.2.0"
+┊    ┊2016┊    upper-case "^1.1.3"
+┊    ┊2017┊
+┊    ┊2018┊hosted-git-info@^2.1.4:
+┊    ┊2019┊  version "2.7.1"
+┊    ┊2020┊  resolved "https://registry.yarnpkg.com/hosted-git-info/-/hosted-git-info-2.7.1.tgz#97f236977bd6e125408930ff6de3eec6281ec047"
+┊    ┊2021┊  integrity sha512-7T/BxH19zbcCTa8XkMlbK5lTo1WtgkFi3GvdWEyNuc4Vex7/9Dqbnpsf4JMydcfj9HCg4zUWFTL3Za6lapg5/w==
+┊    ┊2022┊
 ┊1232┊2023┊http-errors@1.6.3, http-errors@~1.6.2, http-errors@~1.6.3:
 ┊1233┊2024┊  version "1.6.3"
 ┊1234┊2025┊  resolved "https://registry.yarnpkg.com/http-errors/-/http-errors-1.6.3.tgz#8b55680bb4be283a0b5bf4ea2e38580be1d9320d"
```
```diff
@@ -1250,6 +2041,15 @@
 ┊1250┊2041┊    statuses ">= 1.5.0 < 2"
 ┊1251┊2042┊    toidentifier "1.0.0"
 ┊1252┊2043┊
+┊    ┊2044┊http-signature@~1.2.0:
+┊    ┊2045┊  version "1.2.0"
+┊    ┊2046┊  resolved "https://registry.yarnpkg.com/http-signature/-/http-signature-1.2.0.tgz#9aecd925114772f3d95b65a60abb8f7c18fbace1"
+┊    ┊2047┊  integrity sha1-muzZJRFHcvPZW2WmCruPfBj7rOE=
+┊    ┊2048┊  dependencies:
+┊    ┊2049┊    assert-plus "^1.0.0"
+┊    ┊2050┊    jsprim "^1.2.2"
+┊    ┊2051┊    sshpk "^1.7.0"
+┊    ┊2052┊
 ┊1253┊2053┊iconv-lite@0.4.23:
 ┊1254┊2054┊  version "0.4.23"
 ┊1255┊2055┊  resolved "https://registry.yarnpkg.com/iconv-lite/-/iconv-lite-0.4.23.tgz#297871f63be507adcfbfca715d0cd0eed84e9a63"
```
```diff
@@ -1257,7 +2057,7 @@
 ┊1257┊2057┊  dependencies:
 ┊1258┊2058┊    safer-buffer ">= 2.1.2 < 3"
 ┊1259┊2059┊
-┊1260┊    ┊iconv-lite@^0.4.4:
+┊    ┊2060┊iconv-lite@^0.4.24, iconv-lite@^0.4.4:
 ┊1261┊2061┊  version "0.4.24"
 ┊1262┊2062┊  resolved "https://registry.yarnpkg.com/iconv-lite/-/iconv-lite-0.4.24.tgz#2022b4b25fbddc21d2f524974a474aafe733908b"
 ┊1263┊2063┊  integrity sha512-v3MXnZAcvnywkTUEZomIActle7RXXeedOR31wwl7VlyoXO4Qi9arvSenNQWne1TcRwhCL1HwLI21bEqdpj8/rA==
```
```diff
@@ -1276,6 +2076,13 @@
 ┊1276┊2076┊  dependencies:
 ┊1277┊2077┊    minimatch "^3.0.4"
 ┊1278┊2078┊
+┊    ┊2079┊import-from@2.1.0:
+┊    ┊2080┊  version "2.1.0"
+┊    ┊2081┊  resolved "https://registry.yarnpkg.com/import-from/-/import-from-2.1.0.tgz#335db7f2a7affd53aaa471d4b8021dee36b7f3b1"
+┊    ┊2082┊  integrity sha1-M1238qev/VOqpHHUuAId7ja387E=
+┊    ┊2083┊  dependencies:
+┊    ┊2084┊    resolve-from "^3.0.0"
+┊    ┊2085┊
 ┊1279┊2086┊import-lazy@^2.1.0:
 ┊1280┊2087┊  version "2.1.0"
 ┊1281┊2088┊  resolved "https://registry.yarnpkg.com/import-lazy/-/import-lazy-2.1.0.tgz#05698e3d45c88e8d7e9d92cb0584e77f096f3e43"
```
```diff
@@ -1286,6 +2093,11 @@
 ┊1286┊2093┊  resolved "https://registry.yarnpkg.com/imurmurhash/-/imurmurhash-0.1.4.tgz#9218b9b2b928a238b13dc4fb6b6d576f231453ea"
 ┊1287┊2094┊  integrity sha1-khi5srkoojixPcT7a21XbyMUU+o=
 ┊1288┊2095┊
+┊    ┊2096┊indent-string@3.2.0, indent-string@^3.0.0:
+┊    ┊2097┊  version "3.2.0"
+┊    ┊2098┊  resolved "https://registry.yarnpkg.com/indent-string/-/indent-string-3.2.0.tgz#4a5fd6d27cc332f37e5419a504dbb837105c9289"
+┊    ┊2099┊  integrity sha1-Sl/W0nzDMvN+VBmlBNu4NxBckok=
+┊    ┊2100┊
 ┊1289┊2101┊inflight@^1.0.4:
 ┊1290┊2102┊  version "1.0.6"
 ┊1291┊2103┊  resolved "https://registry.yarnpkg.com/inflight/-/inflight-1.0.6.tgz#49bd6331d7d02d0c09bc910a1075ba8165b56df9"
```
```diff
@@ -1304,6 +2116,30 @@
 ┊1304┊2116┊  resolved "https://registry.yarnpkg.com/ini/-/ini-1.3.5.tgz#eee25f56db1c9ec6085e0c22778083f596abf927"
 ┊1305┊2117┊  integrity sha512-RZY5huIKCMRWDUqZlEi72f/lmXKMvuszcMBduliQ3nnWbx9X/ZBQO7DijMEYS9EhHBb2qacRUMtC7svLwe0lcw==
 ┊1306┊2118┊
+┊    ┊2119┊inquirer@6.2.2:
+┊    ┊2120┊  version "6.2.2"
+┊    ┊2121┊  resolved "https://registry.yarnpkg.com/inquirer/-/inquirer-6.2.2.tgz#46941176f65c9eb20804627149b743a218f25406"
+┊    ┊2122┊  integrity sha512-Z2rREiXA6cHRR9KBOarR3WuLlFzlIfAEIiB45ll5SSadMg7WqOh1MKEjjndfuH5ewXdixWCxqnVfGOQzPeiztA==
+┊    ┊2123┊  dependencies:
+┊    ┊2124┊    ansi-escapes "^3.2.0"
+┊    ┊2125┊    chalk "^2.4.2"
+┊    ┊2126┊    cli-cursor "^2.1.0"
+┊    ┊2127┊    cli-width "^2.0.0"
+┊    ┊2128┊    external-editor "^3.0.3"
+┊    ┊2129┊    figures "^2.0.0"
+┊    ┊2130┊    lodash "^4.17.11"
+┊    ┊2131┊    mute-stream "0.0.7"
+┊    ┊2132┊    run-async "^2.2.0"
+┊    ┊2133┊    rxjs "^6.4.0"
+┊    ┊2134┊    string-width "^2.1.0"
+┊    ┊2135┊    strip-ansi "^5.0.0"
+┊    ┊2136┊    through "^2.3.6"
+┊    ┊2137┊
+┊    ┊2138┊invert-kv@^2.0.0:
+┊    ┊2139┊  version "2.0.0"
+┊    ┊2140┊  resolved "https://registry.yarnpkg.com/invert-kv/-/invert-kv-2.0.0.tgz#7393f5afa59ec9ff5f67a27620d11c226e3eec02"
+┊    ┊2141┊  integrity sha512-wPVv/y/QQ/Uiirj/vh3oP+1Ww+AWehmi1g5fFWGPF6IpCBCDVrhgHRMvrLfdYcwDh3QJbGXDW4JAuzxElLSqKA==
+┊    ┊2142┊
 ┊1307┊2143┊ipaddr.js@1.8.0:
 ┊1308┊2144┊  version "1.8.0"
 ┊1309┊2145┊  resolved "https://registry.yarnpkg.com/ipaddr.js/-/ipaddr.js-1.8.0.tgz#eaa33d6ddd7ace8f7f6fe0c9ca0440e706738b1e"
```
```diff
@@ -1323,6 +2159,16 @@
 ┊1323┊2159┊  dependencies:
 ┊1324┊2160┊    kind-of "^6.0.0"
 ┊1325┊2161┊
+┊    ┊2162┊is-arrayish@^0.2.1:
+┊    ┊2163┊  version "0.2.1"
+┊    ┊2164┊  resolved "https://registry.yarnpkg.com/is-arrayish/-/is-arrayish-0.2.1.tgz#77c99840527aa8ecb1a8ba697b80645a7a926a9d"
+┊    ┊2165┊  integrity sha1-d8mYQFJ6qOyxqLppe4BkWnqSap0=
+┊    ┊2166┊
+┊    ┊2167┊is-arrayish@^0.3.1:
+┊    ┊2168┊  version "0.3.2"
+┊    ┊2169┊  resolved "https://registry.yarnpkg.com/is-arrayish/-/is-arrayish-0.3.2.tgz#4574a2ae56f7ab206896fb431eaeed066fdf8f03"
+┊    ┊2170┊  integrity sha512-eVRqCvVlZbuw3GrM63ovNSNAeA1K16kaR/LRY/92w0zxQ5/1YzwblUX652i4Xs9RwAGjW9d9y6X88t8OaAJfWQ==
+┊    ┊2171┊
 ┊1326┊2172┊is-binary-path@^1.0.0:
 ┊1327┊2173┊  version "1.0.1"
 ┊1328┊2174┊  resolved "https://registry.yarnpkg.com/is-binary-path/-/is-binary-path-1.0.1.tgz#75f16642b480f187a711c814161fd3a4a7655898"
```
```diff
@@ -1396,6 +2242,11 @@
 ┊1396┊2242┊  dependencies:
 ┊1397┊2243┊    is-plain-object "^2.0.4"
 ┊1398┊2244┊
+┊    ┊2245┊is-extglob@^1.0.0:
+┊    ┊2246┊  version "1.0.0"
+┊    ┊2247┊  resolved "https://registry.yarnpkg.com/is-extglob/-/is-extglob-1.0.0.tgz#ac468177c4943405a092fc8f29760c6ffc6206c0"
+┊    ┊2248┊  integrity sha1-rEaBd8SUNAWgkvyPKXYMb/xiBsA=
+┊    ┊2249┊
 ┊1399┊2250┊is-extglob@^2.1.0, is-extglob@^2.1.1:
 ┊1400┊2251┊  version "2.1.1"
 ┊1401┊2252┊  resolved "https://registry.yarnpkg.com/is-extglob/-/is-extglob-2.1.1.tgz#a88c02535791f02ed37c76a1b9ea9773c833f8c2"
```
```diff
@@ -1413,6 +2264,20 @@
 ┊1413┊2264┊  resolved "https://registry.yarnpkg.com/is-fullwidth-code-point/-/is-fullwidth-code-point-2.0.0.tgz#a3b30a5c4f199183167aaab93beefae3ddfb654f"
 ┊1414┊2265┊  integrity sha1-o7MKXE8ZkYMWeqq5O+764937ZU8=
 ┊1415┊2266┊
+┊    ┊2267┊is-glob@4.0.0, is-glob@^4.0.0:
+┊    ┊2268┊  version "4.0.0"
+┊    ┊2269┊  resolved "https://registry.yarnpkg.com/is-glob/-/is-glob-4.0.0.tgz#9521c76845cc2610a85203ddf080a958c2ffabc0"
+┊    ┊2270┊  integrity sha1-lSHHaEXMJhCoUgPd8ICpWML/q8A=
+┊    ┊2271┊  dependencies:
+┊    ┊2272┊    is-extglob "^2.1.1"
+┊    ┊2273┊
+┊    ┊2274┊is-glob@^2.0.0:
+┊    ┊2275┊  version "2.0.1"
+┊    ┊2276┊  resolved "https://registry.yarnpkg.com/is-glob/-/is-glob-2.0.1.tgz#d096f926a3ded5600f3fdfd91198cb0888c2d863"
+┊    ┊2277┊  integrity sha1-0Jb5JqPe1WAPP9/ZEZjLCIjC2GM=
+┊    ┊2278┊  dependencies:
+┊    ┊2279┊    is-extglob "^1.0.0"
+┊    ┊2280┊
 ┊1416┊2281┊is-glob@^3.1.0:
 ┊1417┊2282┊  version "3.1.0"
 ┊1418┊2283┊  resolved "https://registry.yarnpkg.com/is-glob/-/is-glob-3.1.0.tgz#7ba5ae24217804ac70707b96922567486cc3e84a"
```
```diff
@@ -1420,13 +2285,6 @@
 ┊1420┊2285┊  dependencies:
 ┊1421┊2286┊    is-extglob "^2.1.0"
 ┊1422┊2287┊
-┊1423┊    ┊is-glob@^4.0.0:
-┊1424┊    ┊  version "4.0.0"
-┊1425┊    ┊  resolved "https://registry.yarnpkg.com/is-glob/-/is-glob-4.0.0.tgz#9521c76845cc2610a85203ddf080a958c2ffabc0"
-┊1426┊    ┊  integrity sha1-lSHHaEXMJhCoUgPd8ICpWML/q8A=
-┊1427┊    ┊  dependencies:
-┊1428┊    ┊    is-extglob "^2.1.1"
-┊1429┊    ┊
 ┊1430┊2288┊is-installed-globally@^0.1.0:
 ┊1431┊2289┊  version "0.1.0"
 ┊1432┊2290┊  resolved "https://registry.yarnpkg.com/is-installed-globally/-/is-installed-globally-0.1.0.tgz#0dfd98f5a9111716dd535dda6492f67bf3d25a80"
```
```diff
@@ -1435,6 +2293,20 @@
 ┊1435┊2293┊    global-dirs "^0.1.0"
 ┊1436┊2294┊    is-path-inside "^1.0.0"
 ┊1437┊2295┊
+┊    ┊2296┊is-invalid-path@^0.1.0:
+┊    ┊2297┊  version "0.1.0"
+┊    ┊2298┊  resolved "https://registry.yarnpkg.com/is-invalid-path/-/is-invalid-path-0.1.0.tgz#307a855b3cf1a938b44ea70d2c61106053714f34"
+┊    ┊2299┊  integrity sha1-MHqFWzzxqTi0TqcNLGEQYFNxTzQ=
+┊    ┊2300┊  dependencies:
+┊    ┊2301┊    is-glob "^2.0.0"
+┊    ┊2302┊
+┊    ┊2303┊is-lower-case@^1.1.0:
+┊    ┊2304┊  version "1.1.3"
+┊    ┊2305┊  resolved "https://registry.yarnpkg.com/is-lower-case/-/is-lower-case-1.1.3.tgz#7e147be4768dc466db3bfb21cc60b31e6ad69393"
+┊    ┊2306┊  integrity sha1-fhR75HaNxGbbO/shzGCzHmrWk5M=
+┊    ┊2307┊  dependencies:
+┊    ┊2308┊    lower-case "^1.1.0"
+┊    ┊2309┊
 ┊1438┊2310┊is-npm@^1.0.0:
 ┊1439┊2311┊  version "1.0.0"
 ┊1440┊2312┊  resolved "https://registry.yarnpkg.com/is-npm/-/is-npm-1.0.0.tgz#f2fb63a65e4905b406c86072765a1a4dc793b9f4"
```
```diff
@@ -1452,6 +2324,13 @@
 ┊1452┊2324┊  resolved "https://registry.yarnpkg.com/is-obj/-/is-obj-1.0.1.tgz#3e4729ac1f5fde025cd7d83a896dab9f4f67db0f"
 ┊1453┊2325┊  integrity sha1-PkcprB9f3gJc19g6iW2rn09n2w8=
 ┊1454┊2326┊
+┊    ┊2327┊is-observable@^1.1.0:
+┊    ┊2328┊  version "1.1.0"
+┊    ┊2329┊  resolved "https://registry.yarnpkg.com/is-observable/-/is-observable-1.1.0.tgz#b3e986c8f44de950867cab5403f5a3465005975e"
+┊    ┊2330┊  integrity sha512-NqCa4Sa2d+u7BWc6CukaObG3Fh+CU9bvixbpcXYhy2VvYS7vVGIdAgnIS5Ks3A/cqk4rebLJ9s8zBstT2aKnIA==
+┊    ┊2331┊  dependencies:
+┊    ┊2332┊    symbol-observable "^1.1.0"
+┊    ┊2333┊
 ┊1455┊2334┊is-path-inside@^1.0.0:
 ┊1456┊2335┊  version "1.0.1"
 ┊1457┊2336┊  resolved "https://registry.yarnpkg.com/is-path-inside/-/is-path-inside-1.0.1.tgz#8ef5b7de50437a3fdca6b4e865ef7aa55cb48036"
```
```diff
@@ -1466,6 +2345,11 @@
 ┊1466┊2345┊  dependencies:
 ┊1467┊2346┊    isobject "^3.0.1"
 ┊1468┊2347┊
+┊    ┊2348┊is-promise@^2.1.0:
+┊    ┊2349┊  version "2.1.0"
+┊    ┊2350┊  resolved "https://registry.yarnpkg.com/is-promise/-/is-promise-2.1.0.tgz#79a2a9ece7f096e80f36d2b2f3bc16c1ff4bf3fa"
+┊    ┊2351┊  integrity sha1-eaKp7OfwlugPNtKy87wWwf9L8/o=
+┊    ┊2352┊
 ┊1469┊2353┊is-redirect@^1.0.0:
 ┊1470┊2354┊  version "1.0.0"
 ┊1471┊2355┊  resolved "https://registry.yarnpkg.com/is-redirect/-/is-redirect-1.0.0.tgz#1d03dded53bd8db0f30c26e4f95d36fc7c87dc24"
```
```diff
@@ -1495,6 +2379,25 @@
 ┊1495┊2379┊  dependencies:
 ┊1496┊2380┊    has-symbols "^1.0.0"
 ┊1497┊2381┊
+┊    ┊2382┊is-typedarray@~1.0.0:
+┊    ┊2383┊  version "1.0.0"
+┊    ┊2384┊  resolved "https://registry.yarnpkg.com/is-typedarray/-/is-typedarray-1.0.0.tgz#e479c80858df0c1b11ddda6940f96011fcda4a9a"
+┊    ┊2385┊  integrity sha1-5HnICFjfDBsR3dppQPlgEfzaSpo=
+┊    ┊2386┊
+┊    ┊2387┊is-upper-case@^1.1.0:
+┊    ┊2388┊  version "1.1.2"
+┊    ┊2389┊  resolved "https://registry.yarnpkg.com/is-upper-case/-/is-upper-case-1.1.2.tgz#8d0b1fa7e7933a1e58483600ec7d9661cbaf756f"
+┊    ┊2390┊  integrity sha1-jQsfp+eTOh5YSDYA7H2WYcuvdW8=
+┊    ┊2391┊  dependencies:
+┊    ┊2392┊    upper-case "^1.1.0"
+┊    ┊2393┊
+┊    ┊2394┊is-valid-path@0.1.1:
+┊    ┊2395┊  version "0.1.1"
+┊    ┊2396┊  resolved "https://registry.yarnpkg.com/is-valid-path/-/is-valid-path-0.1.1.tgz#110f9ff74c37f663e1ec7915eb451f2db93ac9df"
+┊    ┊2397┊  integrity sha1-EQ+f90w39mPh7HkV60UfLbk6yd8=
+┊    ┊2398┊  dependencies:
+┊    ┊2399┊    is-invalid-path "^0.1.0"
+┊    ┊2400┊
 ┊1498┊2401┊is-windows@^1.0.2:
 ┊1499┊2402┊  version "1.0.2"
 ┊1500┊2403┊  resolved "https://registry.yarnpkg.com/is-windows/-/is-windows-1.0.2.tgz#d1850eb9791ecd18e6182ce12a30f396634bb19d"
```
```diff
@@ -1522,11 +2425,77 @@
 ┊1522┊2425┊  resolved "https://registry.yarnpkg.com/isobject/-/isobject-3.0.1.tgz#4e431e92b11a9731636aa1f9c8d1ccbcfdab78df"
 ┊1523┊2426┊  integrity sha1-TkMekrEalzFjaqH5yNHMvP2reN8=
 ┊1524┊2427┊
+┊    ┊2428┊isstream@~0.1.2:
+┊    ┊2429┊  version "0.1.2"
+┊    ┊2430┊  resolved "https://registry.yarnpkg.com/isstream/-/isstream-0.1.2.tgz#47e63f7af55afa6f92e1500e690eb8b8529c099a"
+┊    ┊2431┊  integrity sha1-R+Y/evVa+m+S4VAOaQ64uFKcCZo=
+┊    ┊2432┊
 ┊1525┊2433┊iterall@^1.1.3, iterall@^1.2.1, iterall@^1.2.2:
 ┊1526┊2434┊  version "1.2.2"
 ┊1527┊2435┊  resolved "https://registry.yarnpkg.com/iterall/-/iterall-1.2.2.tgz#92d70deb8028e0c39ff3164fdbf4d8b088130cd7"
 ┊1528┊2436┊  integrity sha512-yynBb1g+RFUPY64fTrFv7nsjRrENBQJaX2UL+2Szc9REFrSNm1rpSXHGzhmAy7a9uv3vlvgBlXnf9RqmPH1/DA==
 ┊1529┊2437┊
+┊    ┊2438┊js-tokens@^4.0.0:
+┊    ┊2439┊  version "4.0.0"
+┊    ┊2440┊  resolved "https://registry.yarnpkg.com/js-tokens/-/js-tokens-4.0.0.tgz#19203fb59991df98e3a287050d4647cdeaf32499"
+┊    ┊2441┊  integrity sha512-RdJUflcE3cUzKiMqQgsCu06FPu9UdIJO0beYbPhHN4k6apgJtifcoCtT9bcxOpYBtpD2kCM6Sbzg4CausW/PKQ==
+┊    ┊2442┊
+┊    ┊2443┊js-yaml@3.12.1, js-yaml@^3.10.0:
+┊    ┊2444┊  version "3.12.1"
+┊    ┊2445┊  resolved "https://registry.yarnpkg.com/js-yaml/-/js-yaml-3.12.1.tgz#295c8632a18a23e054cf5c9d3cecafe678167600"
+┊    ┊2446┊  integrity sha512-um46hB9wNOKlwkHgiuyEVAybXBjwFUV0Z/RaHJblRd9DXltue9FTYvzCr9ErQrK9Adz5MU4gHWVaNUfdmrC8qA==
+┊    ┊2447┊  dependencies:
+┊    ┊2448┊    argparse "^1.0.7"
+┊    ┊2449┊    esprima "^4.0.0"
+┊    ┊2450┊
+┊    ┊2451┊jsbn@~0.1.0:
+┊    ┊2452┊  version "0.1.1"
+┊    ┊2453┊  resolved "https://registry.yarnpkg.com/jsbn/-/jsbn-0.1.1.tgz#a5e654c2e5a2deb5f201d96cefbca80c0ef2f513"
+┊    ┊2454┊  integrity sha1-peZUwuWi3rXyAdls77yoDA7y9RM=
+┊    ┊2455┊
+┊    ┊2456┊jsesc@^2.5.1:
+┊    ┊2457┊  version "2.5.2"
+┊    ┊2458┊  resolved "https://registry.yarnpkg.com/jsesc/-/jsesc-2.5.2.tgz#80564d2e483dacf6e8ef209650a67df3f0c283a4"
+┊    ┊2459┊  integrity sha512-OYu7XEzjkCQ3C5Ps3QIZsQfNpqoJyZZA99wd9aWd05NCtC5pWOkShK2mkL6HXQR6/Cy2lbNdPlZBpuQHXE63gA==
+┊    ┊2460┊
+┊    ┊2461┊json-parse-better-errors@^1.0.1:
+┊    ┊2462┊  version "1.0.2"
+┊    ┊2463┊  resolved "https://registry.yarnpkg.com/json-parse-better-errors/-/json-parse-better-errors-1.0.2.tgz#bb867cfb3450e69107c131d1c514bab3dc8bcaa9"
+┊    ┊2464┊  integrity sha512-mrqyZKfX5EhL7hvqcV6WG1yYjnjeuYDzDhhcAAUrq8Po85NBQBJP+ZDUT75qZQ98IkUoBqdkExkukOU7Ts2wrw==
+┊    ┊2465┊
+┊    ┊2466┊json-schema-traverse@^0.4.1:
+┊    ┊2467┊  version "0.4.1"
+┊    ┊2468┊  resolved "https://registry.yarnpkg.com/json-schema-traverse/-/json-schema-traverse-0.4.1.tgz#69f6a87d9513ab8bb8fe63bdb0979c448e684660"
+┊    ┊2469┊  integrity sha512-xbbCH5dCYU5T8LcEhhuh7HJ88HXuW3qsI3Y0zOZFKfZEHcpWiHU/Jxzk629Brsab/mMiHQti9wMP+845RPe3Vg==
+┊    ┊2470┊
+┊    ┊2471┊json-schema@0.2.3:
+┊    ┊2472┊  version "0.2.3"
+┊    ┊2473┊  resolved "https://registry.yarnpkg.com/json-schema/-/json-schema-0.2.3.tgz#b480c892e59a2f05954ce727bd3f2a4e882f9e13"
+┊    ┊2474┊  integrity sha1-tIDIkuWaLwWVTOcnvT8qTogvnhM=
+┊    ┊2475┊
+┊    ┊2476┊json-stringify-safe@~5.0.1:
+┊    ┊2477┊  version "5.0.1"
+┊    ┊2478┊  resolved "https://registry.yarnpkg.com/json-stringify-safe/-/json-stringify-safe-5.0.1.tgz#1296a2d58fd45f19a0f6ce01d65701e2c735b6eb"
+┊    ┊2479┊  integrity sha1-Epai1Y/UXxmg9s4B1lcB4sc1tus=
+┊    ┊2480┊
+┊    ┊2481┊json-to-pretty-yaml@1.2.2:
+┊    ┊2482┊  version "1.2.2"
+┊    ┊2483┊  resolved "https://registry.yarnpkg.com/json-to-pretty-yaml/-/json-to-pretty-yaml-1.2.2.tgz#f4cd0bd0a5e8fe1df25aaf5ba118b099fd992d5b"
+┊    ┊2484┊  integrity sha1-9M0L0KXo/h3yWq9boRiwmf2ZLVs=
+┊    ┊2485┊  dependencies:
+┊    ┊2486┊    remedial "^1.0.7"
+┊    ┊2487┊    remove-trailing-spaces "^1.0.6"
+┊    ┊2488┊
+┊    ┊2489┊jsprim@^1.2.2:
+┊    ┊2490┊  version "1.4.1"
+┊    ┊2491┊  resolved "https://registry.yarnpkg.com/jsprim/-/jsprim-1.4.1.tgz#313e66bc1e5cc06e438bc1b7499c2e5c56acb6a2"
+┊    ┊2492┊  integrity sha1-MT5mvB5cwG5Di8G3SZwuXFastqI=
+┊    ┊2493┊  dependencies:
+┊    ┊2494┊    assert-plus "1.0.0"
+┊    ┊2495┊    extsprintf "1.3.0"
+┊    ┊2496┊    json-schema "0.2.3"
+┊    ┊2497┊    verror "1.10.0"
+┊    ┊2498┊
 ┊1530┊2499┊kind-of@^3.0.2, kind-of@^3.0.3, kind-of@^3.2.0:
 ┊1531┊2500┊  version "3.2.2"
 ┊1532┊2501┊  resolved "https://registry.yarnpkg.com/kind-of/-/kind-of-3.2.2.tgz#31ea21a734bab9bbb0f32466d893aea51e4a3c64"
```
```diff
@@ -1551,6 +2520,13 @@
 ┊1551┊2520┊  resolved "https://registry.yarnpkg.com/kind-of/-/kind-of-6.0.2.tgz#01146b36a6218e64e58f3a8d66de5d7fc6f6d051"
 ┊1552┊2521┊  integrity sha512-s5kLOcnH0XqDO+FvuaLX8DDjZ18CGFk7VygH40QoKPUQhW4e2rvM0rwUq0t8IQDOwYSeLK01U90OjzBTme2QqA==
 ┊1553┊2522┊
+┊    ┊2523┊kuler@1.0.x:
+┊    ┊2524┊  version "1.0.1"
+┊    ┊2525┊  resolved "https://registry.yarnpkg.com/kuler/-/kuler-1.0.1.tgz#ef7c784f36c9fb6e16dd3150d152677b2b0228a6"
+┊    ┊2526┊  integrity sha512-J9nVUucG1p/skKul6DU3PUZrhs0LPulNaeUOox0IyXDi8S4CztTHs1gQphhuZmzXG7VOQSf6NJfKuzteQLv9gQ==
+┊    ┊2527┊  dependencies:
+┊    ┊2528┊    colornames "^1.1.1"
+┊    ┊2529┊
 ┊1554┊2530┊latest-version@^3.0.0:
 ┊1555┊2531┊  version "3.1.0"
 ┊1556┊2532┊  resolved "https://registry.yarnpkg.com/latest-version/-/latest-version-3.1.0.tgz#a205383fea322b33b5ae3b18abee0dc2f356ee15"
```
```diff
@@ -1558,16 +2534,126 @@
 ┊1558┊2534┊  dependencies:
 ┊1559┊2535┊    package-json "^4.0.0"
 ┊1560┊2536┊
+┊    ┊2537┊lcid@^2.0.0:
+┊    ┊2538┊  version "2.0.0"
+┊    ┊2539┊  resolved "https://registry.yarnpkg.com/lcid/-/lcid-2.0.0.tgz#6ef5d2df60e52f82eb228a4c373e8d1f397253cf"
+┊    ┊2540┊  integrity sha512-avPEb8P8EGnwXKClwsNUgryVjllcRqtMYa49NTsbQagYuT1DcXnl1915oxWjoyGrXR6zH/Y0Zc96xWsPcoDKeA==
+┊    ┊2541┊  dependencies:
+┊    ┊2542┊    invert-kv "^2.0.0"
+┊    ┊2543┊
+┊    ┊2544┊listr-silent-renderer@^1.1.1:
+┊    ┊2545┊  version "1.1.1"
+┊    ┊2546┊  resolved "https://registry.yarnpkg.com/listr-silent-renderer/-/listr-silent-renderer-1.1.1.tgz#924b5a3757153770bf1a8e3fbf74b8bbf3f9242e"
+┊    ┊2547┊  integrity sha1-kktaN1cVN3C/Go4/v3S4u/P5JC4=
+┊    ┊2548┊
+┊    ┊2549┊listr-update-renderer@0.5.0, listr-update-renderer@^0.5.0:
+┊    ┊2550┊  version "0.5.0"
+┊    ┊2551┊  resolved "https://registry.yarnpkg.com/listr-update-renderer/-/listr-update-renderer-0.5.0.tgz#4ea8368548a7b8aecb7e06d8c95cb45ae2ede6a2"
+┊    ┊2552┊  integrity sha512-tKRsZpKz8GSGqoI/+caPmfrypiaq+OQCbd+CovEC24uk1h952lVj5sC7SqyFUm+OaJ5HN/a1YLt5cit2FMNsFA==
+┊    ┊2553┊  dependencies:
+┊    ┊2554┊    chalk "^1.1.3"
+┊    ┊2555┊    cli-truncate "^0.2.1"
+┊    ┊2556┊    elegant-spinner "^1.0.1"
+┊    ┊2557┊    figures "^1.7.0"
+┊    ┊2558┊    indent-string "^3.0.0"
+┊    ┊2559┊    log-symbols "^1.0.2"
+┊    ┊2560┊    log-update "^2.3.0"
+┊    ┊2561┊    strip-ansi "^3.0.1"
+┊    ┊2562┊
+┊    ┊2563┊listr-verbose-renderer@^0.5.0:
+┊    ┊2564┊  version "0.5.0"
+┊    ┊2565┊  resolved "https://registry.yarnpkg.com/listr-verbose-renderer/-/listr-verbose-renderer-0.5.0.tgz#f1132167535ea4c1261102b9f28dac7cba1e03db"
+┊    ┊2566┊  integrity sha512-04PDPqSlsqIOaaaGZ+41vq5FejI9auqTInicFRndCBgE3bXG8D6W1I+mWhk+1nqbHmyhla/6BUrd5OSiHwKRXw==
+┊    ┊2567┊  dependencies:
+┊    ┊2568┊    chalk "^2.4.1"
+┊    ┊2569┊    cli-cursor "^2.1.0"
+┊    ┊2570┊    date-fns "^1.27.2"
+┊    ┊2571┊    figures "^2.0.0"
+┊    ┊2572┊
+┊    ┊2573┊listr@0.14.3:
+┊    ┊2574┊  version "0.14.3"
+┊    ┊2575┊  resolved "https://registry.yarnpkg.com/listr/-/listr-0.14.3.tgz#2fea909604e434be464c50bddba0d496928fa586"
+┊    ┊2576┊  integrity sha512-RmAl7su35BFd/xoMamRjpIE4j3v+L28o8CT5YhAXQJm1fD+1l9ngXY8JAQRJ+tFK2i5njvi0iRUKV09vPwA0iA==
+┊    ┊2577┊  dependencies:
+┊    ┊2578┊    "@samverschueren/stream-to-observable" "^0.3.0"
+┊    ┊2579┊    is-observable "^1.1.0"
+┊    ┊2580┊    is-promise "^2.1.0"
+┊    ┊2581┊    is-stream "^1.1.0"
+┊    ┊2582┊    listr-silent-renderer "^1.1.1"
+┊    ┊2583┊    listr-update-renderer "^0.5.0"
+┊    ┊2584┊    listr-verbose-renderer "^0.5.0"
+┊    ┊2585┊    p-map "^2.0.0"
+┊    ┊2586┊    rxjs "^6.3.3"
+┊    ┊2587┊
+┊    ┊2588┊locate-path@^3.0.0:
+┊    ┊2589┊  version "3.0.0"
+┊    ┊2590┊  resolved "https://registry.yarnpkg.com/locate-path/-/locate-path-3.0.0.tgz#dbec3b3ab759758071b58fe59fc41871af21400e"
+┊    ┊2591┊  integrity sha512-7AO748wWnIhNqAuaty2ZWHkQHRSNfPVIsPIfwEOWO22AmaoVrWavlOcMR5nzTLNYvp36X220/maaRsrec1G65A==
+┊    ┊2592┊  dependencies:
+┊    ┊2593┊    p-locate "^3.0.0"
+┊    ┊2594┊    path-exists "^3.0.0"
+┊    ┊2595┊
 ┊1561┊2596┊lodash.sortby@^4.7.0:
 ┊1562┊2597┊  version "4.7.0"
 ┊1563┊2598┊  resolved "https://registry.yarnpkg.com/lodash.sortby/-/lodash.sortby-4.7.0.tgz#edd14c824e2cc9c1e0b0a1b42bb5210516a42438"
 ┊1564┊2599┊  integrity sha1-7dFMgk4sycHgsKG0K7UhBRakJDg=
 ┊1565┊2600┊
+┊    ┊2601┊lodash@4.17.11, lodash@^4.17.10, lodash@^4.17.11, lodash@^4.17.4, lodash@^4.2.0:
+┊    ┊2602┊  version "4.17.11"
+┊    ┊2603┊  resolved "https://registry.yarnpkg.com/lodash/-/lodash-4.17.11.tgz#b39ea6229ef607ecd89e2c8df12536891cac9b8d"
+┊    ┊2604┊  integrity sha512-cQKh8igo5QUhZ7lg38DYWAxMvjSAKG0A8wGSVimP07SIUEK2UO+arSRKbRZWtelMtN5V0Hkwh5ryOto/SshYIg==
+┊    ┊2605┊
+┊    ┊2606┊log-symbols@2.2.0:
+┊    ┊2607┊  version "2.2.0"
+┊    ┊2608┊  resolved "https://registry.yarnpkg.com/log-symbols/-/log-symbols-2.2.0.tgz#5740e1c5d6f0dfda4ad9323b5332107ef6b4c40a"
+┊    ┊2609┊  integrity sha512-VeIAFslyIerEJLXHziedo2basKbMKtTw3vfn5IzG0XTjhAVEJyNHnL2p7vc+wBDSdQuUpNw3M2u6xb9QsAY5Eg==
+┊    ┊2610┊  dependencies:
+┊    ┊2611┊    chalk "^2.0.1"
+┊    ┊2612┊
+┊    ┊2613┊log-symbols@^1.0.2:
+┊    ┊2614┊  version "1.0.2"
+┊    ┊2615┊  resolved "https://registry.yarnpkg.com/log-symbols/-/log-symbols-1.0.2.tgz#376ff7b58ea3086a0f09facc74617eca501e1a18"
+┊    ┊2616┊  integrity sha1-N2/3tY6jCGoPCfrMdGF+ylAeGhg=
+┊    ┊2617┊  dependencies:
+┊    ┊2618┊    chalk "^1.0.0"
+┊    ┊2619┊
+┊    ┊2620┊log-update@2.3.0, log-update@^2.3.0:
+┊    ┊2621┊  version "2.3.0"
+┊    ┊2622┊  resolved "https://registry.yarnpkg.com/log-update/-/log-update-2.3.0.tgz#88328fd7d1ce7938b29283746f0b1bc126b24708"
+┊    ┊2623┊  integrity sha1-iDKP19HOeTiykoN0bwsbwSayRwg=
+┊    ┊2624┊  dependencies:
+┊    ┊2625┊    ansi-escapes "^3.0.0"
+┊    ┊2626┊    cli-cursor "^2.0.0"
+┊    ┊2627┊    wrap-ansi "^3.0.1"
+┊    ┊2628┊
+┊    ┊2629┊logform@^2.1.1:
+┊    ┊2630┊  version "2.1.2"
+┊    ┊2631┊  resolved "https://registry.yarnpkg.com/logform/-/logform-2.1.2.tgz#957155ebeb67a13164069825ce67ddb5bb2dd360"
+┊    ┊2632┊  integrity sha512-+lZh4OpERDBLqjiwDLpAWNQu6KMjnlXH2ByZwCuSqVPJletw0kTWJf5CgSNAUKn1KUkv3m2cUz/LK8zyEy7wzQ==
+┊    ┊2633┊  dependencies:
+┊    ┊2634┊    colors "^1.2.1"
+┊    ┊2635┊    fast-safe-stringify "^2.0.4"
+┊    ┊2636┊    fecha "^2.3.3"
+┊    ┊2637┊    ms "^2.1.1"
+┊    ┊2638┊    triple-beam "^1.3.0"
+┊    ┊2639┊
 ┊1566┊2640┊long@^4.0.0:
 ┊1567┊2641┊  version "4.0.0"
 ┊1568┊2642┊  resolved "https://registry.yarnpkg.com/long/-/long-4.0.0.tgz#9a7b71cfb7d361a194ea555241c92f7468d5bf28"
 ┊1569┊2643┊  integrity sha512-XsP+KhQif4bjX1kbuSiySJFNAehNxgLb6hPRGJ9QsUr8ajHkuXGdrHmFUTUUXhDwVX2R5bY4JNZEwbUiMhV+MA==
 ┊1570┊2644┊
+┊    ┊2645┊lower-case-first@^1.0.0:
+┊    ┊2646┊  version "1.0.2"
+┊    ┊2647┊  resolved "https://registry.yarnpkg.com/lower-case-first/-/lower-case-first-1.0.2.tgz#e5da7c26f29a7073be02d52bac9980e5922adfa1"
+┊    ┊2648┊  integrity sha1-5dp8JvKacHO+AtUrrJmA5ZIq36E=
+┊    ┊2649┊  dependencies:
+┊    ┊2650┊    lower-case "^1.1.2"
+┊    ┊2651┊
+┊    ┊2652┊lower-case@^1.1.0, lower-case@^1.1.1, lower-case@^1.1.2:
+┊    ┊2653┊  version "1.1.4"
+┊    ┊2654┊  resolved "https://registry.yarnpkg.com/lower-case/-/lower-case-1.1.4.tgz#9a2cabd1b9e8e0ae993a4bf7d5875c39c42e8eac"
+┊    ┊2655┊  integrity sha1-miyr0bno4K6ZOkv31YdcOcQujqw=
+┊    ┊2656┊
 ┊1571┊2657┊lowercase-keys@^1.0.0:
 ┊1572┊2658┊  version "1.0.1"
 ┊1573┊2659┊  resolved "https://registry.yarnpkg.com/lowercase-keys/-/lowercase-keys-1.0.1.tgz#6f9e30b47084d971a7c820ff15a6c5167b74c26f"
```
```diff
@@ -1600,6 +2686,13 @@
 ┊1600┊2686┊  resolved "https://registry.yarnpkg.com/make-error/-/make-error-1.3.5.tgz#efe4e81f6db28cadd605c70f29c831b58ef776c8"
 ┊1601┊2687┊  integrity sha512-c3sIjNUow0+8swNwVpqoH4YCShKNFkMaw6oH1mNS2haDZQqkeZFlHS3dhoeEbKKmJB4vXpJucU6oH75aDYeE9g==
 ┊1602┊2688┊
+┊    ┊2689┊map-age-cleaner@^0.1.1:
+┊    ┊2690┊  version "0.1.3"
+┊    ┊2691┊  resolved "https://registry.yarnpkg.com/map-age-cleaner/-/map-age-cleaner-0.1.3.tgz#7d583a7306434c055fe474b0f45078e6e1b4b92a"
+┊    ┊2692┊  integrity sha512-bJzx6nMoP6PDLPBFmg7+xRKeFZvFboMrGlxmNj9ClvX53KrmvM5bXFXEWjbz4cz1AFn+jWJ9z/DJSz7hrs0w3w==
+┊    ┊2693┊  dependencies:
+┊    ┊2694┊    p-defer "^1.0.0"
+┊    ┊2695┊
 ┊1603┊2696┊map-cache@^0.2.2:
 ┊1604┊2697┊  version "0.2.2"
 ┊1605┊2698┊  resolved "https://registry.yarnpkg.com/map-cache/-/map-cache-0.2.2.tgz#c32abd0bd6525d9b051645bb4f26ac5dc98a0dbf"
```
```diff
@@ -1617,6 +2710,15 @@
 ┊1617┊2710┊  resolved "https://registry.yarnpkg.com/media-typer/-/media-typer-0.3.0.tgz#8710d7af0aa626f8fffa1ce00168545263255748"
 ┊1618┊2711┊  integrity sha1-hxDXrwqmJvj/+hzgAWhUUmMlV0g=
 ┊1619┊2712┊
+┊    ┊2713┊mem@^4.0.0:
+┊    ┊2714┊  version "4.1.0"
+┊    ┊2715┊  resolved "https://registry.yarnpkg.com/mem/-/mem-4.1.0.tgz#aeb9be2d21f47e78af29e4ac5978e8afa2ca5b8a"
+┊    ┊2716┊  integrity sha512-I5u6Q1x7wxO0kdOpYBB28xueHADYps5uty/zg936CiG8NTe5sJL8EjrCuLneuDW3PlMdZBGDIn8BirEVdovZvg==
+┊    ┊2717┊  dependencies:
+┊    ┊2718┊    map-age-cleaner "^0.1.1"
+┊    ┊2719┊    mimic-fn "^1.0.0"
+┊    ┊2720┊    p-is-promise "^2.0.0"
+┊    ┊2721┊
 ┊1620┊2722┊merge-descriptors@1.0.1:
 ┊1621┊2723┊  version "1.0.1"
 ┊1622┊2724┊  resolved "https://registry.yarnpkg.com/merge-descriptors/-/merge-descriptors-1.0.1.tgz#b00aaa556dd8b44568150ec9d1b953f3f90cbb61"
```
```diff
@@ -1651,7 +2753,7 @@
 ┊1651┊2753┊  resolved "https://registry.yarnpkg.com/mime-db/-/mime-db-1.38.0.tgz#1a2aab16da9eb167b49c6e4df2d9c68d63d8e2ad"
 ┊1652┊2754┊  integrity sha512-bqVioMFFzc2awcdJZIzR3HjZFX20QhilVS7hytkKrv7xFAn8bM1gzc/FOX2awLISvWe0PV8ptFKcon+wZ5qYkg==
 ┊1653┊2755┊
-┊1654┊    ┊mime-types@~2.1.18:
+┊    ┊2756┊mime-types@^2.1.12, mime-types@~2.1.18, mime-types@~2.1.19:
 ┊1655┊2757┊  version "2.1.22"
 ┊1656┊2758┊  resolved "https://registry.yarnpkg.com/mime-types/-/mime-types-2.1.22.tgz#fe6b355a190926ab7698c9a0556a11199b2199bd"
 ┊1657┊2759┊  integrity sha512-aGl6TZGnhm/li6F7yx82bJiBZwgiEa4Hf6CNr8YO+r5UHr53tSTYZb102zyU50DOWWKeOv0uQLRL0/9EiKWCog==
```
```diff
@@ -1663,6 +2765,11 @@
 ┊1663┊2765┊  resolved "https://registry.yarnpkg.com/mime/-/mime-1.4.1.tgz#121f9ebc49e3766f311a76e1fa1c8003c4b03aa6"
 ┊1664┊2766┊  integrity sha512-KI1+qOZu5DcW6wayYHSzR/tXKCDC5Om4s1z2QJjDULzLcmf3DvzS7oluY4HCTrc+9FiKmWUgeNLg7W3uIQvxtQ==
 ┊1665┊2767┊
+┊    ┊2768┊mimic-fn@^1.0.0:
+┊    ┊2769┊  version "1.2.0"
+┊    ┊2770┊  resolved "https://registry.yarnpkg.com/mimic-fn/-/mimic-fn-1.2.0.tgz#820c86a39334640e99516928bd03fca88057d022"
+┊    ┊2771┊  integrity sha512-jf84uxzwiuiIVKiOLpfYk7N46TSy8ubTonmneY9vrpHNAnp0QBt2BxWV9dO3/j+BoVAb+a5G6YDPW3M5HOdMWQ==
+┊    ┊2772┊
 ┊1666┊2773┊minimatch@^3.0.4:
 ┊1667┊2774┊  version "3.0.4"
 ┊1668┊2775┊  resolved "https://registry.yarnpkg.com/minimatch/-/minimatch-3.0.4.tgz#5166e286457f03306064be5497e8dbb0c3d32083"
```
```diff
@@ -1703,7 +2810,7 @@
 ┊1703┊2810┊    for-in "^1.0.2"
 ┊1704┊2811┊    is-extendable "^1.0.1"
 ┊1705┊2812┊
-┊1706┊    ┊mkdirp@^0.5.0, mkdirp@^0.5.1:
+┊    ┊2813┊mkdirp@0.5.1, mkdirp@^0.5.0, mkdirp@^0.5.1:
 ┊1707┊2814┊  version "0.5.1"
 ┊1708┊2815┊  resolved "https://registry.yarnpkg.com/mkdirp/-/mkdirp-0.5.1.tgz#30057438eac6cf7f8c4767f38648d6697d75c903"
 ┊1709┊2816┊  integrity sha1-MAV0OOrGz3+MR2fzhkjWaX11yQM=
```
```diff
@@ -1725,6 +2832,11 @@
 ┊1725┊2832┊  resolved "https://registry.yarnpkg.com/ms/-/ms-2.1.1.tgz#30a5864eb3ebb0a66f2ebe6d727af06a09d86e0a"
 ┊1726┊2833┊  integrity sha512-tgp+dl5cGk28utYktBsrFqA7HKgrhgPsg6Z/EfhWI4gl1Hwq8B/GmY/0oXZ6nF8hDVesS/FpnYaD/kOWhYQvyg==
 ┊1727┊2834┊
+┊    ┊2835┊mute-stream@0.0.7:
+┊    ┊2836┊  version "0.0.7"
+┊    ┊2837┊  resolved "https://registry.yarnpkg.com/mute-stream/-/mute-stream-0.0.7.tgz#3075ce93bc21b8fab43e1bc4da7e8115ed1e7bab"
+┊    ┊2838┊  integrity sha1-MHXOk7whuPq0PhvE2n6BFe0ee6s=
+┊    ┊2839┊
 ┊1728┊2840┊nan@^2.9.2:
 ┊1729┊2841┊  version "2.12.1"
 ┊1730┊2842┊  resolved "https://registry.yarnpkg.com/nan/-/nan-2.12.1.tgz#7b1aa193e9aa86057e3c7bbd0ac448e770925552"
```
```diff
@@ -1761,6 +2873,23 @@
 ┊1761┊2873┊  resolved "https://registry.yarnpkg.com/negotiator/-/negotiator-0.6.1.tgz#2b327184e8992101177b28563fb5e7102acd0ca9"
 ┊1762┊2874┊  integrity sha1-KzJxhOiZIQEXeyhWP7XnECrNDKk=
 ┊1763┊2875┊
+┊    ┊2876┊nice-try@^1.0.4:
+┊    ┊2877┊  version "1.0.5"
+┊    ┊2878┊  resolved "https://registry.yarnpkg.com/nice-try/-/nice-try-1.0.5.tgz#a3378a7696ce7d223e88fc9b764bd7ef1089e366"
+┊    ┊2879┊  integrity sha512-1nh45deeb5olNY7eX82BkPO7SSxR5SSYJiPTrTdFUVYwAl8CKMA5N9PjTYkHiRjisVcxcQ1HXdLhx2qxxJzLNQ==
+┊    ┊2880┊
+┊    ┊2881┊no-case@^2.2.0, no-case@^2.3.2:
+┊    ┊2882┊  version "2.3.2"
+┊    ┊2883┊  resolved "https://registry.yarnpkg.com/no-case/-/no-case-2.3.2.tgz#60b813396be39b3f1288a4c1ed5d1e7d28b464ac"
+┊    ┊2884┊  integrity sha512-rmTZ9kz+f3rCvK2TD1Ue/oZlns7OGoIWP4fc3llxxRXlOkHKoWPPWJOfFYpITabSow43QJbRIoHQXtt10VldyQ==
+┊    ┊2885┊  dependencies:
+┊    ┊2886┊    lower-case "^1.1.1"
+┊    ┊2887┊
+┊    ┊2888┊node-fetch@2.1.2:
+┊    ┊2889┊  version "2.1.2"
+┊    ┊2890┊  resolved "https://registry.yarnpkg.com/node-fetch/-/node-fetch-2.1.2.tgz#ab884e8e7e57e38a944753cec706f788d1768bb5"
+┊    ┊2891┊  integrity sha1-q4hOjn5X44qUR1POxwb3iNF2i7U=
+┊    ┊2892┊
 ┊1764┊2893┊node-fetch@^2.1.2, node-fetch@^2.2.0:
 ┊1765┊2894┊  version "2.3.0"
 ┊1766┊2895┊  resolved "https://registry.yarnpkg.com/node-fetch/-/node-fetch-2.3.0.tgz#1a1d940bbfb916a1d3e0219f037e89e71f8c5fa5"
```
```diff
@@ -1813,6 +2942,16 @@
 ┊1813┊2942┊  dependencies:
 ┊1814┊2943┊    abbrev "1"
 ┊1815┊2944┊
+┊    ┊2945┊normalize-package-data@^2.3.2:
+┊    ┊2946┊  version "2.5.0"
+┊    ┊2947┊  resolved "https://registry.yarnpkg.com/normalize-package-data/-/normalize-package-data-2.5.0.tgz#e66db1838b200c1dfc233225d12cb36520e234a8"
+┊    ┊2948┊  integrity sha512-/5CMN3T0R4XTj4DcGaexo+roZSdSFW/0AOOTROrjxzCG1wrWXEsGbRKevjlIL+ZDE4sZlJr5ED4YW0yqmkK+eA==
+┊    ┊2949┊  dependencies:
+┊    ┊2950┊    hosted-git-info "^2.1.4"
+┊    ┊2951┊    resolve "^1.10.0"
+┊    ┊2952┊    semver "2 || 3 || 4 || 5"
+┊    ┊2953┊    validate-npm-package-license "^3.0.1"
+┊    ┊2954┊
 ┊1816┊2955┊normalize-path@^2.1.1:
 ┊1817┊2956┊  version "2.1.1"
 ┊1818┊2957┊  resolved "https://registry.yarnpkg.com/normalize-path/-/normalize-path-2.1.1.tgz#1ab28b556e198363a8c1a6f7e6fa20137fe6aed9"
```
```diff
@@ -1860,6 +2999,11 @@
 ┊1860┊2999┊  resolved "https://registry.yarnpkg.com/number-is-nan/-/number-is-nan-1.0.1.tgz#097b602b53422a522c1afb8790318336941a011d"
 ┊1861┊3000┊  integrity sha1-CXtgK1NCKlIsGvuHkDGDNpQaAR0=
 ┊1862┊3001┊
+┊    ┊3002┊oauth-sign@~0.9.0:
+┊    ┊3003┊  version "0.9.0"
+┊    ┊3004┊  resolved "https://registry.yarnpkg.com/oauth-sign/-/oauth-sign-0.9.0.tgz#47a7b016baa68b5fa0ecf3dee08a85c679ac6455"
+┊    ┊3005┊  integrity sha512-fexhUFFPTGV8ybAtSIGbV6gOkSv8UtRbDBnAyLQw4QPKkgNlsH2ByPGtMUqdWkos6YCRmAqViwgZrJc/mRDzZQ==
+┊    ┊3006┊
 ┊1863┊3007┊object-assign@^4, object-assign@^4.1.0:
 ┊1864┊3008┊  version "4.1.1"
 ┊1865┊3009┊  resolved "https://registry.yarnpkg.com/object-assign/-/object-assign-4.1.1.tgz#2109adc7965887cfc05cbbd442cac8bfbb360863"
```
```diff
@@ -1913,19 +3057,40 @@
 ┊1913┊3057┊  dependencies:
 ┊1914┊3058┊    ee-first "1.1.1"
 ┊1915┊3059┊
-┊1916┊    ┊once@^1.3.0:
+┊    ┊3060┊once@^1.3.0, once@^1.3.1, once@^1.4.0:
 ┊1917┊3061┊  version "1.4.0"
 ┊1918┊3062┊  resolved "https://registry.yarnpkg.com/once/-/once-1.4.0.tgz#583b1aa775961d4b113ac17d9c50baef9dd76bd1"
 ┊1919┊3063┊  integrity sha1-WDsap3WWHUsROsF9nFC6753Xa9E=
 ┊1920┊3064┊  dependencies:
 ┊1921┊3065┊    wrappy "1"
 ┊1922┊3066┊
+┊    ┊3067┊one-time@0.0.4:
+┊    ┊3068┊  version "0.0.4"
+┊    ┊3069┊  resolved "https://registry.yarnpkg.com/one-time/-/one-time-0.0.4.tgz#f8cdf77884826fe4dff93e3a9cc37b1e4480742e"
+┊    ┊3070┊  integrity sha1-+M33eISCb+Tf+T46nMN7HkSAdC4=
+┊    ┊3071┊
+┊    ┊3072┊onetime@^2.0.0:
+┊    ┊3073┊  version "2.0.1"
+┊    ┊3074┊  resolved "https://registry.yarnpkg.com/onetime/-/onetime-2.0.1.tgz#067428230fd67443b2794b22bba528b6867962d4"
+┊    ┊3075┊  integrity sha1-BnQoIw/WdEOyeUsiu6UotoZ5YtQ=
+┊    ┊3076┊  dependencies:
+┊    ┊3077┊    mimic-fn "^1.0.0"
+┊    ┊3078┊
 ┊1923┊3079┊os-homedir@^1.0.0:
 ┊1924┊3080┊  version "1.0.2"
 ┊1925┊3081┊  resolved "https://registry.yarnpkg.com/os-homedir/-/os-homedir-1.0.2.tgz#ffbc4988336e0e833de0c168c7ef152121aa7fb3"
 ┊1926┊3082┊  integrity sha1-/7xJiDNuDoM94MFox+8VISGqf7M=
 ┊1927┊3083┊
-┊1928┊    ┊os-tmpdir@^1.0.0:
+┊    ┊3084┊os-locale@^3.0.0:
+┊    ┊3085┊  version "3.1.0"
+┊    ┊3086┊  resolved "https://registry.yarnpkg.com/os-locale/-/os-locale-3.1.0.tgz#a802a6ee17f24c10483ab9935719cef4ed16bf1a"
+┊    ┊3087┊  integrity sha512-Z8l3R4wYWM40/52Z+S265okfFj8Kt2cC2MKY+xNi3kFs+XGI7WXu/I309QQQYbRW4ijiZ+yxs9pqEhJh0DqW3Q==
+┊    ┊3088┊  dependencies:
+┊    ┊3089┊    execa "^1.0.0"
+┊    ┊3090┊    lcid "^2.0.0"
+┊    ┊3091┊    mem "^4.0.0"
+┊    ┊3092┊
+┊    ┊3093┊os-tmpdir@^1.0.0, os-tmpdir@~1.0.2:
 ┊1929┊3094┊  version "1.0.2"
 ┊1930┊3095┊  resolved "https://registry.yarnpkg.com/os-tmpdir/-/os-tmpdir-1.0.2.tgz#bbe67406c79aa85c5cfec766fe5734555dfa1274"
 ┊1931┊3096┊  integrity sha1-u+Z0BseaqFxc/sdm/lc0VV36EnQ=
```
```diff
@@ -1938,11 +3103,45 @@
 ┊1938┊3103┊    os-homedir "^1.0.0"
 ┊1939┊3104┊    os-tmpdir "^1.0.0"
 ┊1940┊3105┊
+┊    ┊3106┊p-defer@^1.0.0:
+┊    ┊3107┊  version "1.0.0"
+┊    ┊3108┊  resolved "https://registry.yarnpkg.com/p-defer/-/p-defer-1.0.0.tgz#9f6eb182f6c9aa8cd743004a7d4f96b196b0fb0c"
+┊    ┊3109┊  integrity sha1-n26xgvbJqozXQwBKfU+WsZaw+ww=
+┊    ┊3110┊
 ┊1941┊3111┊p-finally@^1.0.0:
 ┊1942┊3112┊  version "1.0.0"
 ┊1943┊3113┊  resolved "https://registry.yarnpkg.com/p-finally/-/p-finally-1.0.0.tgz#3fbcfb15b899a44123b34b6dcc18b724336a2cae"
 ┊1944┊3114┊  integrity sha1-P7z7FbiZpEEjs0ttzBi3JDNqLK4=
 ┊1945┊3115┊
+┊    ┊3116┊p-is-promise@^2.0.0:
+┊    ┊3117┊  version "2.0.0"
+┊    ┊3118┊  resolved "https://registry.yarnpkg.com/p-is-promise/-/p-is-promise-2.0.0.tgz#7554e3d572109a87e1f3f53f6a7d85d1b194f4c5"
+┊    ┊3119┊  integrity sha512-pzQPhYMCAgLAKPWD2jC3Se9fEfrD9npNos0y150EeqZll7akhEgGhTW/slB6lHku8AvYGiJ+YJ5hfHKePPgFWg==
+┊    ┊3120┊
+┊    ┊3121┊p-limit@^2.0.0:
+┊    ┊3122┊  version "2.1.0"
+┊    ┊3123┊  resolved "https://registry.yarnpkg.com/p-limit/-/p-limit-2.1.0.tgz#1d5a0d20fb12707c758a655f6bbc4386b5930d68"
+┊    ┊3124┊  integrity sha512-NhURkNcrVB+8hNfLuysU8enY5xn2KXphsHBaC2YmRNTZRc7RWusw6apSpdEj3jo4CMb6W9nrF6tTnsJsJeyu6g==
+┊    ┊3125┊  dependencies:
+┊    ┊3126┊    p-try "^2.0.0"
+┊    ┊3127┊
+┊    ┊3128┊p-locate@^3.0.0:
+┊    ┊3129┊  version "3.0.0"
+┊    ┊3130┊  resolved "https://registry.yarnpkg.com/p-locate/-/p-locate-3.0.0.tgz#322d69a05c0264b25997d9f40cd8a891ab0064a4"
+┊    ┊3131┊  integrity sha512-x+12w/To+4GFfgJhBEpiDcLozRJGegY+Ei7/z0tSLkMmxGZNybVMSfWj9aJn8Z5Fc7dBUNJOOVgPv2H7IwulSQ==
+┊    ┊3132┊  dependencies:
+┊    ┊3133┊    p-limit "^2.0.0"
+┊    ┊3134┊
+┊    ┊3135┊p-map@^2.0.0:
+┊    ┊3136┊  version "2.0.0"
+┊    ┊3137┊  resolved "https://registry.yarnpkg.com/p-map/-/p-map-2.0.0.tgz#be18c5a5adeb8e156460651421aceca56c213a50"
+┊    ┊3138┊  integrity sha512-GO107XdrSUmtHxVoi60qc9tUl/KkNKm+X2CF4P9amalpGxv5YqVPJNfSb0wcA+syCopkZvYYIzW8OVTQW59x/w==
+┊    ┊3139┊
+┊    ┊3140┊p-try@^2.0.0:
+┊    ┊3141┊  version "2.0.0"
+┊    ┊3142┊  resolved "https://registry.yarnpkg.com/p-try/-/p-try-2.0.0.tgz#85080bb87c64688fa47996fe8f7dfbe8211760b1"
+┊    ┊3143┊  integrity sha512-hMp0onDKIajHfIkdRk3P4CdCmErkYAxxDtP3Wx/4nZ3aGlau2VKh3mZpcuFkH27WQkL/3WBCPOktzA9ZOAnMQQ==
+┊    ┊3144┊
 ┊1946┊3145┊package-json@^4.0.0:
 ┊1947┊3146┊  version "4.0.1"
 ┊1948┊3147┊  resolved "https://registry.yarnpkg.com/package-json/-/package-json-4.0.1.tgz#8869a0401253661c4c4ca3da6c2121ed555f5eed"
```
```diff
@@ -1953,21 +3152,56 @@
 ┊1953┊3152┊    registry-url "^3.0.3"
 ┊1954┊3153┊    semver "^5.1.0"
 ┊1955┊3154┊
+┊    ┊3155┊param-case@^2.1.0:
+┊    ┊3156┊  version "2.1.1"
+┊    ┊3157┊  resolved "https://registry.yarnpkg.com/param-case/-/param-case-2.1.1.tgz#df94fd8cf6531ecf75e6bef9a0858fbc72be2247"
+┊    ┊3158┊  integrity sha1-35T9jPZTHs915r75oIWPvHK+Ikc=
+┊    ┊3159┊  dependencies:
+┊    ┊3160┊    no-case "^2.2.0"
+┊    ┊3161┊
+┊    ┊3162┊parse-json@^4.0.0:
+┊    ┊3163┊  version "4.0.0"
+┊    ┊3164┊  resolved "https://registry.yarnpkg.com/parse-json/-/parse-json-4.0.0.tgz#be35f5425be1f7f6c747184f98a788cb99477ee0"
+┊    ┊3165┊  integrity sha1-vjX1Qlvh9/bHRxhPmKeIy5lHfuA=
+┊    ┊3166┊  dependencies:
+┊    ┊3167┊    error-ex "^1.3.1"
+┊    ┊3168┊    json-parse-better-errors "^1.0.1"
+┊    ┊3169┊
 ┊1956┊3170┊parseurl@~1.3.2:
 ┊1957┊3171┊  version "1.3.2"
 ┊1958┊3172┊  resolved "https://registry.yarnpkg.com/parseurl/-/parseurl-1.3.2.tgz#fc289d4ed8993119460c156253262cdc8de65bf3"
 ┊1959┊3173┊  integrity sha1-/CidTtiZMRlGDBViUyYs3I3mW/M=
 ┊1960┊3174┊
+┊    ┊3175┊pascal-case@^2.0.0:
+┊    ┊3176┊  version "2.0.1"
+┊    ┊3177┊  resolved "https://registry.yarnpkg.com/pascal-case/-/pascal-case-2.0.1.tgz#2d578d3455f660da65eca18ef95b4e0de912761e"
+┊    ┊3178┊  integrity sha1-LVeNNFX2YNpl7KGO+VtODekSdh4=
+┊    ┊3179┊  dependencies:
+┊    ┊3180┊    camel-case "^3.0.0"
+┊    ┊3181┊    upper-case-first "^1.1.0"
+┊    ┊3182┊
 ┊1961┊3183┊pascalcase@^0.1.1:
 ┊1962┊3184┊  version "0.1.1"
 ┊1963┊3185┊  resolved "https://registry.yarnpkg.com/pascalcase/-/pascalcase-0.1.1.tgz#b363e55e8006ca6fe21784d2db22bd15d7917f14"
 ┊1964┊3186┊  integrity sha1-s2PlXoAGym/iF4TS2yK9FdeRfxQ=
 ┊1965┊3187┊
+┊    ┊3188┊path-case@^2.1.0:
+┊    ┊3189┊  version "2.1.1"
+┊    ┊3190┊  resolved "https://registry.yarnpkg.com/path-case/-/path-case-2.1.1.tgz#94b8037c372d3fe2906e465bb45e25d226e8eea5"
+┊    ┊3191┊  integrity sha1-lLgDfDctP+KQbkZbtF4l0ibo7qU=
+┊    ┊3192┊  dependencies:
+┊    ┊3193┊    no-case "^2.2.0"
+┊    ┊3194┊
 ┊1966┊3195┊path-dirname@^1.0.0:
 ┊1967┊3196┊  version "1.0.2"
 ┊1968┊3197┊  resolved "https://registry.yarnpkg.com/path-dirname/-/path-dirname-1.0.2.tgz#cc33d24d525e099a5388c0336c6e32b9160609e0"
 ┊1969┊3198┊  integrity sha1-zDPSTVJeCZpTiMAzbG4yuRYGCeA=
 ┊1970┊3199┊
+┊    ┊3200┊path-exists@^3.0.0:
+┊    ┊3201┊  version "3.0.0"
+┊    ┊3202┊  resolved "https://registry.yarnpkg.com/path-exists/-/path-exists-3.0.0.tgz#ce0ebeaa5f78cb18925ea7d810d7b59b010fd515"
+┊    ┊3203┊  integrity sha1-zg6+ql94yxiSXqfYENe1mwEP1RU=
+┊    ┊3204┊
 ┊1971┊3205┊path-is-absolute@^1.0.0:
 ┊1972┊3206┊  version "1.0.1"
 ┊1973┊3207┊  resolved "https://registry.yarnpkg.com/path-is-absolute/-/path-is-absolute-1.0.1.tgz#174b9268735534ffbc7ace6bf53a5a9e1b5c5f5f"
```
```diff
@@ -1978,16 +3212,26 @@
 ┊1978┊3212┊  resolved "https://registry.yarnpkg.com/path-is-inside/-/path-is-inside-1.0.2.tgz#365417dede44430d1c11af61027facf074bdfc53"
 ┊1979┊3213┊  integrity sha1-NlQX3t5EQw0cEa9hAn+s8HS9/FM=
 ┊1980┊3214┊
-┊1981┊    ┊path-key@^2.0.0:
+┊    ┊3215┊path-key@^2.0.0, path-key@^2.0.1:
 ┊1982┊3216┊  version "2.0.1"
 ┊1983┊3217┊  resolved "https://registry.yarnpkg.com/path-key/-/path-key-2.0.1.tgz#411cadb574c5a140d3a4b1910d40d80cc9f40b40"
 ┊1984┊3218┊  integrity sha1-QRyttXTFoUDTpLGRDUDYDMn0C0A=
 ┊1985┊3219┊
+┊    ┊3220┊path-parse@^1.0.6:
+┊    ┊3221┊  version "1.0.6"
+┊    ┊3222┊  resolved "https://registry.yarnpkg.com/path-parse/-/path-parse-1.0.6.tgz#d62dbb5679405d72c4737ec58600e9ddcf06d24c"
+┊    ┊3223┊  integrity sha512-GSmOT2EbHrINBf9SR7CDELwlJ8AENk3Qn7OikK4nFYAu3Ote2+JYNVvkpAEQm3/TLNEJFD/xZJjzyxg3KBWOzw==
+┊    ┊3224┊
 ┊1986┊3225┊path-to-regexp@0.1.7:
 ┊1987┊3226┊  version "0.1.7"
 ┊1988┊3227┊  resolved "https://registry.yarnpkg.com/path-to-regexp/-/path-to-regexp-0.1.7.tgz#df604178005f522f15eb4490e7247a1bfaa67f8c"
 ┊1989┊3228┊  integrity sha1-32BBeABfUi8V60SQ5yR6G/qmf4w=
 ┊1990┊3229┊
+┊    ┊3230┊performance-now@^2.1.0:
+┊    ┊3231┊  version "2.1.0"
+┊    ┊3232┊  resolved "https://registry.yarnpkg.com/performance-now/-/performance-now-2.1.0.tgz#6309f4e0e5fa913ec1c69307ae364b4b377c9e7b"
+┊    ┊3233┊  integrity sha1-Ywn04OX6kT7BxpMHrjZLSzd8nns=
+┊    ┊3234┊
 ┊1991┊3235┊pify@^3.0.0:
 ┊1992┊3236┊  version "3.0.0"
 ┊1993┊3237┊  resolved "https://registry.yarnpkg.com/pify/-/pify-3.0.0.tgz#e5a4acd2c101fdf3d9a4d07f0dbc4db49dd28176"
```
```diff
@@ -2003,6 +3247,11 @@
 ┊2003┊3247┊  resolved "https://registry.yarnpkg.com/prepend-http/-/prepend-http-1.0.4.tgz#d4f4562b0ce3696e41ac52d0e002e57a635dc6dc"
 ┊2004┊3248┊  integrity sha1-1PRWKwzjaW5BrFLQ4ALlemNdxtw=
 ┊2005┊3249┊
+┊    ┊3250┊prettier@1.16.4:
+┊    ┊3251┊  version "1.16.4"
+┊    ┊3252┊  resolved "https://registry.yarnpkg.com/prettier/-/prettier-1.16.4.tgz#73e37e73e018ad2db9c76742e2647e21790c9717"
+┊    ┊3253┊  integrity sha512-ZzWuos7TI5CKUeQAtFd6Zhm2s6EpAD/ZLApIhsF9pRvRtM1RFo61dM/4MSRUA0SuLugA/zgrZD8m0BaY46Og7g==
+┊    ┊3254┊
 ┊2006┊3255┊process-nextick-args@~2.0.0:
 ┊2007┊3256┊  version "2.0.0"
 ┊2008┊3257┊  resolved "https://registry.yarnpkg.com/process-nextick-args/-/process-nextick-args-2.0.0.tgz#a37d732f4271b4ab1ad070d35508e8290788ffaa"
```
```diff
@@ -2040,12 +3289,35 @@
 ┊2040┊3289┊  resolved "https://registry.yarnpkg.com/pseudomap/-/pseudomap-1.0.2.tgz#f052a28da70e618917ef0a8ac34c1ae5a68286b3"
 ┊2041┊3290┊  integrity sha1-8FKijacOYYkX7wqKw0wa5aaChrM=
 ┊2042┊3291┊
+┊    ┊3292┊psl@^1.1.24:
+┊    ┊3293┊  version "1.1.31"
+┊    ┊3294┊  resolved "https://registry.yarnpkg.com/psl/-/psl-1.1.31.tgz#e9aa86d0101b5b105cbe93ac6b784cd547276184"
+┊    ┊3295┊  integrity sha512-/6pt4+C+T+wZUieKR620OpzN/LlnNKuWjy1iFLQ/UG35JqHlR/89MP1d96dUfkf6Dne3TuLQzOYEYshJ+Hx8mw==
+┊    ┊3296┊
 ┊2043┊3297┊pstree.remy@^1.1.6:
 ┊2044┊3298┊  version "1.1.6"
 ┊2045┊3299┊  resolved "https://registry.yarnpkg.com/pstree.remy/-/pstree.remy-1.1.6.tgz#73a55aad9e2d95814927131fbf4dc1b62d259f47"
 ┊2046┊3300┊  integrity sha512-NdF35+QsqD7EgNEI5mkI/X+UwaxVEbQaz9f4IooEmMUv6ZPmlTQYGjBPJGgrlzNdjSvIy4MWMg6Q6vCgBO2K+w==
 ┊2047┊3301┊
-┊2048┊    ┊qs@6.5.2:
+┊    ┊3302┊pump@^3.0.0:
+┊    ┊3303┊  version "3.0.0"
+┊    ┊3304┊  resolved "https://registry.yarnpkg.com/pump/-/pump-3.0.0.tgz#b4a2116815bde2f4e1ea602354e8c75565107a64"
+┊    ┊3305┊  integrity sha512-LwZy+p3SFs1Pytd/jYct4wpv49HiYCqd9Rlc5ZVdk0V+8Yzv6jR5Blk3TRmPL1ft69TxP0IMZGJ+WPFU2BFhww==
+┊    ┊3306┊  dependencies:
+┊    ┊3307┊    end-of-stream "^1.1.0"
+┊    ┊3308┊    once "^1.3.1"
+┊    ┊3309┊
+┊    ┊3310┊punycode@^1.4.1:
+┊    ┊3311┊  version "1.4.1"
+┊    ┊3312┊  resolved "https://registry.yarnpkg.com/punycode/-/punycode-1.4.1.tgz#c0d5a63b2718800ad8e1eb0fa5269c84dd41845e"
+┊    ┊3313┊  integrity sha1-wNWmOycYgArY4esPpSachN1BhF4=
+┊    ┊3314┊
+┊    ┊3315┊punycode@^2.1.0:
+┊    ┊3316┊  version "2.1.1"
+┊    ┊3317┊  resolved "https://registry.yarnpkg.com/punycode/-/punycode-2.1.1.tgz#b58b010ac40c22c5657616c8d2c2c02c7bf479ec"
+┊    ┊3318┊  integrity sha512-XRsRjdf+j5ml+y/6GKHPZbrF/8p2Yga0JPtdqTIY2Xe5ohJPD9saDJJLPvp9+NSBprVvevdXZybnj2cv8OEd0A==
+┊    ┊3319┊
+┊    ┊3320┊qs@6.5.2, qs@~6.5.2:
 ┊2049┊3321┊  version "6.5.2"
 ┊2050┊3322┊  resolved "https://registry.yarnpkg.com/qs/-/qs-6.5.2.tgz#cb3ae806e8740444584ef154ce8ee98d403f3e36"
 ┊2051┊3323┊  integrity sha512-N5ZAX4/LxJmF+7wN74pUD6qAh9/wnvdQcjq9TZjevvXzSUo7bfmw91saqMjzGS2xq91/odN2dW/WOl7qQHNDGA==
```
```diff
@@ -2075,7 +3347,16 @@
 ┊2075┊3347┊    minimist "^1.2.0"
 ┊2076┊3348┊    strip-json-comments "~2.0.1"
 ┊2077┊3349┊
-┊2078┊    ┊readable-stream@^2.0.2, readable-stream@^2.0.6:
+┊    ┊3350┊read-pkg@^4.0.1:
+┊    ┊3351┊  version "4.0.1"
+┊    ┊3352┊  resolved "https://registry.yarnpkg.com/read-pkg/-/read-pkg-4.0.1.tgz#963625378f3e1c4d48c85872b5a6ec7d5d093237"
+┊    ┊3353┊  integrity sha1-ljYlN48+HE1IyFhytabsfV0JMjc=
+┊    ┊3354┊  dependencies:
+┊    ┊3355┊    normalize-package-data "^2.3.2"
+┊    ┊3356┊    parse-json "^4.0.0"
+┊    ┊3357┊    pify "^3.0.0"
+┊    ┊3358┊
+┊    ┊3359┊readable-stream@^2.0.2, readable-stream@^2.0.6, readable-stream@^2.3.6:
 ┊2079┊3360┊  version "2.3.6"
 ┊2080┊3361┊  resolved "https://registry.yarnpkg.com/readable-stream/-/readable-stream-2.3.6.tgz#b11c27d88b8ff1fbe070643cf94b0c79ae1b0aaf"
 ┊2081┊3362┊  integrity sha512-tQtKA9WIAhBF3+VLAseyMqZeBjW0AHJoxOtYqSUZNJxauErmLbVm2FW1y+J/YA9dUrAC39ITejlZWhVIwawkKw==
```
```diff
@@ -2088,6 +3369,15 @@
 ┊2088┊3369┊    string_decoder "~1.1.1"
 ┊2089┊3370┊    util-deprecate "~1.0.1"
 ┊2090┊3371┊
+┊    ┊3372┊readable-stream@^3.1.1:
+┊    ┊3373┊  version "3.1.1"
+┊    ┊3374┊  resolved "https://registry.yarnpkg.com/readable-stream/-/readable-stream-3.1.1.tgz#ed6bbc6c5ba58b090039ff18ce670515795aeb06"
+┊    ┊3375┊  integrity sha512-DkN66hPyqDhnIQ6Jcsvx9bFjhw214O4poMBcIMgPVpQvNy9a0e0Uhg5SqySyDKAmUlwt8LonTBz1ezOnM8pUdA==
+┊    ┊3376┊  dependencies:
+┊    ┊3377┊    inherits "^2.0.3"
+┊    ┊3378┊    string_decoder "^1.1.1"
+┊    ┊3379┊    util-deprecate "^1.0.1"
+┊    ┊3380┊
 ┊2091┊3381┊readdirp@^2.2.1:
 ┊2092┊3382┊  version "2.2.1"
 ┊2093┊3383┊  resolved "https://registry.yarnpkg.com/readdirp/-/readdirp-2.2.1.tgz#0e87622a3325aa33e892285caf8b4e846529a525"
```
```diff
@@ -2120,11 +3410,21 @@
 ┊2120┊3410┊  dependencies:
 ┊2121┊3411┊    rc "^1.0.1"
 ┊2122┊3412┊
+┊    ┊3413┊remedial@^1.0.7:
+┊    ┊3414┊  version "1.0.8"
+┊    ┊3415┊  resolved "https://registry.yarnpkg.com/remedial/-/remedial-1.0.8.tgz#a5e4fd52a0e4956adbaf62da63a5a46a78c578a0"
+┊    ┊3416┊  integrity sha512-/62tYiOe6DzS5BqVsNpH/nkGlX45C/Sp6V+NtiN6JQNS1Viay7cWkazmRkrQrdFj2eshDe96SIQNIoMxqhzBOg==
+┊    ┊3417┊
 ┊2123┊3418┊remove-trailing-separator@^1.0.1:
 ┊2124┊3419┊  version "1.1.0"
 ┊2125┊3420┊  resolved "https://registry.yarnpkg.com/remove-trailing-separator/-/remove-trailing-separator-1.1.0.tgz#c24bce2a283adad5bc3f58e0d48249b92379d8ef"
 ┊2126┊3421┊  integrity sha1-wkvOKig62tW8P1jg1IJJuSN52O8=
 ┊2127┊3422┊
+┊    ┊3423┊remove-trailing-spaces@^1.0.6:
+┊    ┊3424┊  version "1.0.7"
+┊    ┊3425┊  resolved "https://registry.yarnpkg.com/remove-trailing-spaces/-/remove-trailing-spaces-1.0.7.tgz#491f04e11d98880714d12429b0d0938cbe030ae6"
+┊    ┊3426┊  integrity sha512-wjM17CJ2kk0SgoGyJ7ZMzRRCuTq+V8YhMwpZ5XEWX0uaked2OUq6utvHXGNBQrfkUzUUABFMyxlKn+85hMv4dg==
+┊    ┊3427┊
 ┊2128┊3428┊repeat-element@^1.1.2:
 ┊2129┊3429┊  version "1.1.3"
 ┊2130┊3430┊  resolved "https://registry.yarnpkg.com/repeat-element/-/repeat-element-1.1.3.tgz#782e0d825c0c5a3bb39731f84efee6b742e6b1ce"
```
```diff
@@ -2135,11 +3435,72 @@
 ┊2135┊3435┊  resolved "https://registry.yarnpkg.com/repeat-string/-/repeat-string-1.6.1.tgz#8dcae470e1c88abc2d600fff4a776286da75e637"
 ┊2136┊3436┊  integrity sha1-jcrkcOHIirwtYA//Sndihtp15jc=
 ┊2137┊3437┊
+┊    ┊3438┊request@2.88.0:
+┊    ┊3439┊  version "2.88.0"
+┊    ┊3440┊  resolved "https://registry.yarnpkg.com/request/-/request-2.88.0.tgz#9c2fca4f7d35b592efe57c7f0a55e81052124fef"
+┊    ┊3441┊  integrity sha512-NAqBSrijGLZdM0WZNsInLJpkJokL72XYjUpnB0iwsRgxh7dB6COrHnTBNwN0E+lHDAJzu7kLAkDeY08z2/A0hg==
+┊    ┊3442┊  dependencies:
+┊    ┊3443┊    aws-sign2 "~0.7.0"
+┊    ┊3444┊    aws4 "^1.8.0"
+┊    ┊3445┊    caseless "~0.12.0"
+┊    ┊3446┊    combined-stream "~1.0.6"
+┊    ┊3447┊    extend "~3.0.2"
+┊    ┊3448┊    forever-agent "~0.6.1"
+┊    ┊3449┊    form-data "~2.3.2"
+┊    ┊3450┊    har-validator "~5.1.0"
+┊    ┊3451┊    http-signature "~1.2.0"
+┊    ┊3452┊    is-typedarray "~1.0.0"
+┊    ┊3453┊    isstream "~0.1.2"
+┊    ┊3454┊    json-stringify-safe "~5.0.1"
+┊    ┊3455┊    mime-types "~2.1.19"
+┊    ┊3456┊    oauth-sign "~0.9.0"
+┊    ┊3457┊    performance-now "^2.1.0"
+┊    ┊3458┊    qs "~6.5.2"
+┊    ┊3459┊    safe-buffer "^5.1.2"
+┊    ┊3460┊    tough-cookie "~2.4.3"
+┊    ┊3461┊    tunnel-agent "^0.6.0"
+┊    ┊3462┊    uuid "^3.3.2"
+┊    ┊3463┊
+┊    ┊3464┊require-directory@^2.1.1:
+┊    ┊3465┊  version "2.1.1"
+┊    ┊3466┊  resolved "https://registry.yarnpkg.com/require-directory/-/require-directory-2.1.1.tgz#8c64ad5fd30dab1c976e2344ffe7f792a6a6df42"
+┊    ┊3467┊  integrity sha1-jGStX9MNqxyXbiNE/+f3kqam30I=
+┊    ┊3468┊
+┊    ┊3469┊require-main-filename@^1.0.1:
+┊    ┊3470┊  version "1.0.1"
+┊    ┊3471┊  resolved "https://registry.yarnpkg.com/require-main-filename/-/require-main-filename-1.0.1.tgz#97f717b69d48784f5f526a6c5aa8ffdda055a4d1"
+┊    ┊3472┊  integrity sha1-l/cXtp1IeE9fUmpsWqj/3aBVpNE=
+┊    ┊3473┊
+┊    ┊3474┊resolve-from@^3.0.0:
+┊    ┊3475┊  version "3.0.0"
+┊    ┊3476┊  resolved "https://registry.yarnpkg.com/resolve-from/-/resolve-from-3.0.0.tgz#b22c7af7d9d6881bc8b6e653335eebcb0a188748"
+┊    ┊3477┊  integrity sha1-six699nWiBvItuZTM17rywoYh0g=
+┊    ┊3478┊
+┊    ┊3479┊resolve-from@^4.0.0:
+┊    ┊3480┊  version "4.0.0"
+┊    ┊3481┊  resolved "https://registry.yarnpkg.com/resolve-from/-/resolve-from-4.0.0.tgz#4abcd852ad32dd7baabfe9b40e00a36db5f392e6"
+┊    ┊3482┊  integrity sha512-pb/MYmXstAkysRFx8piNI1tGFNQIFA3vkE3Gq4EuA1dF6gHp/+vgZqsCGJapvy8N3Q+4o7FwvquPJcnZ7RYy4g==
+┊    ┊3483┊
 ┊2138┊3484┊resolve-url@^0.2.1:
 ┊2139┊3485┊  version "0.2.1"
 ┊2140┊3486┊  resolved "https://registry.yarnpkg.com/resolve-url/-/resolve-url-0.2.1.tgz#2c637fe77c893afd2a663fe21aa9080068e2052a"
 ┊2141┊3487┊  integrity sha1-LGN/53yJOv0qZj/iGqkIAGjiBSo=
 ┊2142┊3488┊
+┊    ┊3489┊resolve@^1.10.0:
+┊    ┊3490┊  version "1.10.0"
+┊    ┊3491┊  resolved "https://registry.yarnpkg.com/resolve/-/resolve-1.10.0.tgz#3bdaaeaf45cc07f375656dfd2e54ed0810b101ba"
+┊    ┊3492┊  integrity sha512-3sUr9aq5OfSg2S9pNtPA9hL1FVEAjvfOC4leW0SNf/mpnaakz2a9femSd6LqAww2RaFctwyf1lCqnTHuF1rxDg==
+┊    ┊3493┊  dependencies:
+┊    ┊3494┊    path-parse "^1.0.6"
+┊    ┊3495┊
+┊    ┊3496┊restore-cursor@^2.0.0:
+┊    ┊3497┊  version "2.0.0"
+┊    ┊3498┊  resolved "https://registry.yarnpkg.com/restore-cursor/-/restore-cursor-2.0.0.tgz#9f7ee287f82fd326d4fd162923d62129eee0dfaf"
+┊    ┊3499┊  integrity sha1-n37ih/gv0ybU/RYpI9YhKe7g368=
+┊    ┊3500┊  dependencies:
+┊    ┊3501┊    onetime "^2.0.0"
+┊    ┊3502┊    signal-exit "^3.0.2"
+┊    ┊3503┊
 ┊2143┊3504┊ret@~0.1.10:
 ┊2144┊3505┊  version "0.1.15"
 ┊2145┊3506┊  resolved "https://registry.yarnpkg.com/ret/-/ret-0.1.15.tgz#b8a4825d5bdb1fc3f6f53c2bc33f81388681c7bc"
```
```diff
@@ -2157,6 +3518,20 @@
 ┊2157┊3518┊  dependencies:
 ┊2158┊3519┊    glob "^7.1.3"
 ┊2159┊3520┊
+┊    ┊3521┊run-async@^2.2.0:
+┊    ┊3522┊  version "2.3.0"
+┊    ┊3523┊  resolved "https://registry.yarnpkg.com/run-async/-/run-async-2.3.0.tgz#0371ab4ae0bdd720d4166d7dfda64ff7a445a6c0"
+┊    ┊3524┊  integrity sha1-A3GrSuC91yDUFm19/aZP96RFpsA=
+┊    ┊3525┊  dependencies:
+┊    ┊3526┊    is-promise "^2.1.0"
+┊    ┊3527┊
+┊    ┊3528┊rxjs@^6.3.3, rxjs@^6.4.0:
+┊    ┊3529┊  version "6.4.0"
+┊    ┊3530┊  resolved "https://registry.yarnpkg.com/rxjs/-/rxjs-6.4.0.tgz#f3bb0fe7bda7fb69deac0c16f17b50b0b8790504"
+┊    ┊3531┊  integrity sha512-Z9Yfa11F6B9Sg/BK9MnqnQ+aQYicPLtilXBp2yUtDt2JRCE0h26d33EnfO3ZxoNxG0T92OUucP3Ct7cpfkdFfw==
+┊    ┊3532┊  dependencies:
+┊    ┊3533┊    tslib "^1.9.0"
+┊    ┊3534┊
 ┊2160┊3535┊safe-buffer@5.1.2, safe-buffer@^5.0.1, safe-buffer@^5.1.2, safe-buffer@~5.1.0, safe-buffer@~5.1.1:
 ┊2161┊3536┊  version "5.1.2"
 ┊2162┊3537┊  resolved "https://registry.yarnpkg.com/safe-buffer/-/safe-buffer-5.1.2.tgz#991ec69d296e0313747d59bdfd2b745c35f8828d"
```
```diff
@@ -2169,7 +3544,7 @@
 ┊2169┊3544┊  dependencies:
 ┊2170┊3545┊    ret "~0.1.10"
 ┊2171┊3546┊
-┊2172┊    ┊"safer-buffer@>= 2.1.2 < 3":
+┊    ┊3547┊"safer-buffer@>= 2.1.2 < 3", safer-buffer@^2.0.2, safer-buffer@^2.1.0, safer-buffer@~2.1.0:
 ┊2173┊3548┊  version "2.1.2"
 ┊2174┊3549┊  resolved "https://registry.yarnpkg.com/safer-buffer/-/safer-buffer-2.1.2.tgz#44fa161b0187b9549dd84bb91802f9bd8385cd6a"
 ┊2175┊3550┊  integrity sha512-YZo3K82SD7Riyi0E1EQPojLz7kpepnSQI9IyPbHHg1XXXevb5dJI7tpyN2ADxGcQbHG7vcyRHk0cbwqcQriUtg==
```
```diff
@@ -2186,7 +3561,7 @@
 ┊2186┊3561┊  dependencies:
 ┊2187┊3562┊    semver "^5.0.3"
 ┊2188┊3563┊
-┊2189┊    ┊semver@^5.0.3, semver@^5.1.0, semver@^5.3.0, semver@^5.5.0:
+┊    ┊3564┊"semver@2 || 3 || 4 || 5", semver@^5.0.3, semver@^5.1.0, semver@^5.3.0, semver@^5.5.0:
 ┊2190┊3565┊  version "5.6.0"
 ┊2191┊3566┊  resolved "https://registry.yarnpkg.com/semver/-/semver-5.6.0.tgz#7e74256fbaa49c75aa7c7a205cc22799cac80004"
 ┊2192┊3567┊  integrity sha512-RS9R6R35NYgQn++fkDWaOmqGoj4Ek9gGs+DPxNUZKuwE183xjJroKvyo1IzVFeXvUrvmALy6FWD5xrdJT25gMg==
```
```diff
@@ -2210,6 +3585,14 @@
 ┊2210┊3585┊    range-parser "~1.2.0"
 ┊2211┊3586┊    statuses "~1.4.0"
 ┊2212┊3587┊
+┊    ┊3588┊sentence-case@^2.1.0:
+┊    ┊3589┊  version "2.1.1"
+┊    ┊3590┊  resolved "https://registry.yarnpkg.com/sentence-case/-/sentence-case-2.1.1.tgz#1f6e2dda39c168bf92d13f86d4a918933f667ed4"
+┊    ┊3591┊  integrity sha1-H24t2jnBaL+S0T+G1KkYkz9mftQ=
+┊    ┊3592┊  dependencies:
+┊    ┊3593┊    no-case "^2.2.0"
+┊    ┊3594┊    upper-case-first "^1.1.2"
+┊    ┊3595┊
 ┊2213┊3596┊serve-static@1.13.2:
 ┊2214┊3597┊  version "1.13.2"
 ┊2215┊3598┊  resolved "https://registry.yarnpkg.com/serve-static/-/serve-static-1.13.2.tgz#095e8472fd5b46237db50ce486a43f4b86c6cec1"
```
```diff
@@ -2220,7 +3603,7 @@
 ┊2220┊3603┊    parseurl "~1.3.2"
 ┊2221┊3604┊    send "0.16.2"
 ┊2222┊3605┊
-┊2223┊    ┊set-blocking@~2.0.0:
+┊    ┊3606┊set-blocking@^2.0.0, set-blocking@~2.0.0:
 ┊2224┊3607┊  version "2.0.0"
 ┊2225┊3608┊  resolved "https://registry.yarnpkg.com/set-blocking/-/set-blocking-2.0.0.tgz#045f9782d011ae9a6803ddd382b24392b3d890f7"
 ┊2226┊3609┊  integrity sha1-BF+XgtARrppoA93TgrJDkrPYkPc=
```
```diff
@@ -2280,6 +3663,25 @@
 ┊2280┊3663┊  resolved "https://registry.yarnpkg.com/signal-exit/-/signal-exit-3.0.2.tgz#b5fdc08f1287ea1178628e415e25132b73646c6d"
 ┊2281┊3664┊  integrity sha1-tf3AjxKH6hF4Yo5BXiUTK3NkbG0=
 ┊2282┊3665┊
+┊    ┊3666┊simple-swizzle@^0.2.2:
+┊    ┊3667┊  version "0.2.2"
+┊    ┊3668┊  resolved "https://registry.yarnpkg.com/simple-swizzle/-/simple-swizzle-0.2.2.tgz#a4da6b635ffcccca33f70d17cb92592de95e557a"
+┊    ┊3669┊  integrity sha1-pNprY1/8zMoz9w0Xy5JZLeleVXo=
+┊    ┊3670┊  dependencies:
+┊    ┊3671┊    is-arrayish "^0.3.1"
+┊    ┊3672┊
+┊    ┊3673┊slice-ansi@0.0.4:
+┊    ┊3674┊  version "0.0.4"
+┊    ┊3675┊  resolved "https://registry.yarnpkg.com/slice-ansi/-/slice-ansi-0.0.4.tgz#edbf8903f66f7ce2f8eafd6ceed65e264c831b35"
+┊    ┊3676┊  integrity sha1-7b+JA/ZvfOL46v1s7tZeJkyDGzU=
+┊    ┊3677┊
+┊    ┊3678┊snake-case@^2.1.0:
+┊    ┊3679┊  version "2.1.0"
+┊    ┊3680┊  resolved "https://registry.yarnpkg.com/snake-case/-/snake-case-2.1.0.tgz#41bdb1b73f30ec66a04d4e2cad1b76387d4d6d9f"
+┊    ┊3681┊  integrity sha1-Qb2xtz8w7GagTU4srRt2OH1NbZ8=
+┊    ┊3682┊  dependencies:
+┊    ┊3683┊    no-case "^2.2.0"
+┊    ┊3684┊
 ┊2283┊3685┊snapdragon-node@^2.0.1:
 ┊2284┊3686┊  version "2.1.1"
 ┊2285┊3687┊  resolved "https://registry.yarnpkg.com/snapdragon-node/-/snapdragon-node-2.1.1.tgz#6c175f86ff14bdb0724563e8f3c1b021a286853b"
```
```diff
@@ -2321,7 +3723,7 @@
 ┊2321┊3723┊    source-map-url "^0.4.0"
 ┊2322┊3724┊    urix "^0.1.0"
 ┊2323┊3725┊
-┊2324┊    ┊source-map-support@^0.5.6:
+┊    ┊3726┊source-map-support@^0.5.6, source-map-support@^0.5.9:
 ┊2325┊3727┊  version "0.5.10"
 ┊2326┊3728┊  resolved "https://registry.yarnpkg.com/source-map-support/-/source-map-support-0.5.10.tgz#2214080bc9d51832511ee2bab96e3c2f9353120c"
 ┊2327┊3729┊  integrity sha512-YfQ3tQFTK/yzlGJuX8pTwa4tifQj4QS2Mj7UegOu8jAz59MqIiMGPXxQhVQiIMNzayuUSF/jEuVnfFF5JqybmQ==
```
```diff
@@ -2334,7 +3736,7 @@
 ┊2334┊3736┊  resolved "https://registry.yarnpkg.com/source-map-url/-/source-map-url-0.4.0.tgz#3e935d7ddd73631b97659956d55128e87b5084a3"
 ┊2335┊3737┊  integrity sha1-PpNdfd1zYxuXZZlW1VEo6HtQhKM=
 ┊2336┊3738┊
-┊2337┊    ┊source-map@^0.5.6:
+┊    ┊3739┊source-map@^0.5.0, source-map@^0.5.6:
 ┊2338┊3740┊  version "0.5.7"
 ┊2339┊3741┊  resolved "https://registry.yarnpkg.com/source-map/-/source-map-0.5.7.tgz#8a039d2d1021d22d1ea14c80d8ea468ba2ef3fcc"
 ┊2340┊3742┊  integrity sha1-igOdLRAh0i0eoUyA2OpGi6LvP8w=
```
```diff
@@ -2344,6 +3746,37 @@
 ┊2344┊3746┊  resolved "https://registry.yarnpkg.com/source-map/-/source-map-0.6.1.tgz#74722af32e9614e9c287a8d0bbde48b5e2f1a263"
 ┊2345┊3747┊  integrity sha512-UjgapumWlbMhkBgzT7Ykc5YXUT46F0iKu8SGXq0bcwP5dz/h0Plj6enJqjz1Zbq2l5WaqYnrVbwWOWMyF3F47g==
 ┊2346┊3748┊
+┊    ┊3749┊spawn-command@^0.0.2-1:
+┊    ┊3750┊  version "0.0.2-1"
+┊    ┊3751┊  resolved "https://registry.yarnpkg.com/spawn-command/-/spawn-command-0.0.2-1.tgz#62f5e9466981c1b796dc5929937e11c9c6921bd0"
+┊    ┊3752┊  integrity sha1-YvXpRmmBwbeW3Fkpk34RycaSG9A=
+┊    ┊3753┊
+┊    ┊3754┊spdx-correct@^3.0.0:
+┊    ┊3755┊  version "3.1.0"
+┊    ┊3756┊  resolved "https://registry.yarnpkg.com/spdx-correct/-/spdx-correct-3.1.0.tgz#fb83e504445268f154b074e218c87c003cd31df4"
+┊    ┊3757┊  integrity sha512-lr2EZCctC2BNR7j7WzJ2FpDznxky1sjfxvvYEyzxNyb6lZXHODmEoJeFu4JupYlkfha1KZpJyoqiJ7pgA1qq8Q==
+┊    ┊3758┊  dependencies:
+┊    ┊3759┊    spdx-expression-parse "^3.0.0"
+┊    ┊3760┊    spdx-license-ids "^3.0.0"
+┊    ┊3761┊
+┊    ┊3762┊spdx-exceptions@^2.1.0:
+┊    ┊3763┊  version "2.2.0"
+┊    ┊3764┊  resolved "https://registry.yarnpkg.com/spdx-exceptions/-/spdx-exceptions-2.2.0.tgz#2ea450aee74f2a89bfb94519c07fcd6f41322977"
+┊    ┊3765┊  integrity sha512-2XQACfElKi9SlVb1CYadKDXvoajPgBVPn/gOQLrTvHdElaVhr7ZEbqJaRnJLVNeaI4cMEAgVCeBMKF6MWRDCRA==
+┊    ┊3766┊
+┊    ┊3767┊spdx-expression-parse@^3.0.0:
+┊    ┊3768┊  version "3.0.0"
+┊    ┊3769┊  resolved "https://registry.yarnpkg.com/spdx-expression-parse/-/spdx-expression-parse-3.0.0.tgz#99e119b7a5da00e05491c9fa338b7904823b41d0"
+┊    ┊3770┊  integrity sha512-Yg6D3XpRD4kkOmTpdgbUiEJFKghJH03fiC1OPll5h/0sO6neh2jqRDVHOQ4o/LMea0tgCkbMgea5ip/e+MkWyg==
+┊    ┊3771┊  dependencies:
+┊    ┊3772┊    spdx-exceptions "^2.1.0"
+┊    ┊3773┊    spdx-license-ids "^3.0.0"
+┊    ┊3774┊
+┊    ┊3775┊spdx-license-ids@^3.0.0:
+┊    ┊3776┊  version "3.0.3"
+┊    ┊3777┊  resolved "https://registry.yarnpkg.com/spdx-license-ids/-/spdx-license-ids-3.0.3.tgz#81c0ce8f21474756148bbb5f3bfc0f36bf15d76e"
+┊    ┊3778┊  integrity sha512-uBIcIl3Ih6Phe3XHK1NqboJLdGfwr1UN3k6wSD1dZpmPsIkb8AGNbZYJ1fOBk834+Gxy8rpfDxrS6XLEMZMY2g==
+┊    ┊3779┊
 ┊2347┊3780┊split-string@^3.0.1, split-string@^3.0.2:
 ┊2348┊3781┊  version "3.1.0"
 ┊2349┊3782┊  resolved "https://registry.yarnpkg.com/split-string/-/split-string-3.1.0.tgz#7cb09dda3a86585705c64b39a6466038682e8fe2"
```
```diff
@@ -2351,6 +3784,31 @@
 ┊2351┊3784┊  dependencies:
 ┊2352┊3785┊    extend-shallow "^3.0.0"
 ┊2353┊3786┊
+┊    ┊3787┊sprintf-js@~1.0.2:
+┊    ┊3788┊  version "1.0.3"
+┊    ┊3789┊  resolved "https://registry.yarnpkg.com/sprintf-js/-/sprintf-js-1.0.3.tgz#04e6926f662895354f3dd015203633b857297e2c"
+┊    ┊3790┊  integrity sha1-BOaSb2YolTVPPdAVIDYzuFcpfiw=
+┊    ┊3791┊
+┊    ┊3792┊sshpk@^1.7.0:
+┊    ┊3793┊  version "1.16.1"
+┊    ┊3794┊  resolved "https://registry.yarnpkg.com/sshpk/-/sshpk-1.16.1.tgz#fb661c0bef29b39db40769ee39fa70093d6f6877"
+┊    ┊3795┊  integrity sha512-HXXqVUq7+pcKeLqqZj6mHFUMvXtOJt1uoUx09pFW6011inTMxqI8BA8PM95myrIyyKwdnzjdFjLiE6KBPVtJIg==
+┊    ┊3796┊  dependencies:
+┊    ┊3797┊    asn1 "~0.2.3"
+┊    ┊3798┊    assert-plus "^1.0.0"
+┊    ┊3799┊    bcrypt-pbkdf "^1.0.0"
+┊    ┊3800┊    dashdash "^1.12.0"
+┊    ┊3801┊    ecc-jsbn "~0.1.1"
+┊    ┊3802┊    getpass "^0.1.1"
+┊    ┊3803┊    jsbn "~0.1.0"
+┊    ┊3804┊    safer-buffer "^2.0.2"
+┊    ┊3805┊    tweetnacl "~0.14.0"
+┊    ┊3806┊
+┊    ┊3807┊stack-trace@0.0.x:
+┊    ┊3808┊  version "0.0.10"
+┊    ┊3809┊  resolved "https://registry.yarnpkg.com/stack-trace/-/stack-trace-0.0.10.tgz#547c70b347e8d32b4e108ea1a2a159e5fdde19c0"
+┊    ┊3810┊  integrity sha1-VHxws0fo0ytOEI6hoqFZ5f3eGcA=
+┊    ┊3811┊
 ┊2354┊3812┊static-extend@^0.1.1:
 ┊2355┊3813┊  version "0.1.2"
 ┊2356┊3814┊  resolved "https://registry.yarnpkg.com/static-extend/-/static-extend-0.1.2.tgz#60809c39cbff55337226fd5e0b520f341f1fb5c6"
```
```diff
@@ -2383,7 +3841,7 @@
 ┊2383┊3841┊    is-fullwidth-code-point "^1.0.0"
 ┊2384┊3842┊    strip-ansi "^3.0.0"
 ┊2385┊3843┊
-┊2386┊    ┊"string-width@^1.0.2 || 2", string-width@^2.0.0, string-width@^2.1.1:
+┊    ┊3844┊"string-width@^1.0.2 || 2", string-width@^2.0.0, string-width@^2.1.0, string-width@^2.1.1:
 ┊2387┊3845┊  version "2.1.1"
 ┊2388┊3846┊  resolved "https://registry.yarnpkg.com/string-width/-/string-width-2.1.1.tgz#ab93f27a8dc13d28cac815c462143a6d9012ae9e"
 ┊2389┊3847┊  integrity sha512-nOqH59deCq9SRHlxq1Aw85Jnt4w6KvLKqWVik6oA9ZklXLNIOlqg4F2yrT1MVaTjAqvVwdfeZ7w7aCvJD7ugkw==
```
```diff
@@ -2391,6 +3849,13 @@
 ┊2391┊3849┊    is-fullwidth-code-point "^2.0.0"
 ┊2392┊3850┊    strip-ansi "^4.0.0"
 ┊2393┊3851┊
+┊    ┊3852┊string_decoder@^1.1.1:
+┊    ┊3853┊  version "1.2.0"
+┊    ┊3854┊  resolved "https://registry.yarnpkg.com/string_decoder/-/string_decoder-1.2.0.tgz#fe86e738b19544afe70469243b2a1ee9240eae8d"
+┊    ┊3855┊  integrity sha512-6YqyX6ZWEYguAxgZzHGL7SsCeGx3V2TtOTqZz1xSTSWnqsbWwbptafNyvf/ACquZUXV3DANr5BDIwNYe1mN42w==
+┊    ┊3856┊  dependencies:
+┊    ┊3857┊    safe-buffer "~5.1.0"
+┊    ┊3858┊
 ┊2394┊3859┊string_decoder@~1.1.1:
 ┊2395┊3860┊  version "1.1.1"
 ┊2396┊3861┊  resolved "https://registry.yarnpkg.com/string_decoder/-/string_decoder-1.1.1.tgz#9cf1611ba62685d7030ae9e4ba34149c3af03fc8"
```
```diff
@@ -2412,6 +3877,13 @@
 ┊2412┊3877┊  dependencies:
 ┊2413┊3878┊    ansi-regex "^3.0.0"
 ┊2414┊3879┊
+┊    ┊3880┊strip-ansi@^5.0.0:
+┊    ┊3881┊  version "5.0.0"
+┊    ┊3882┊  resolved "https://registry.yarnpkg.com/strip-ansi/-/strip-ansi-5.0.0.tgz#f78f68b5d0866c20b2c9b8c61b5298508dc8756f"
+┊    ┊3883┊  integrity sha512-Uu7gQyZI7J7gn5qLn1Np3G9vcYGTVqB+lFTytnDJv83dd8T22aGH451P3jueT2/QemInJDfxHB5Tde5OzgG1Ow==
+┊    ┊3884┊  dependencies:
+┊    ┊3885┊    ansi-regex "^4.0.0"
+┊    ┊3886┊
 ┊2415┊3887┊strip-eof@^1.0.0:
 ┊2416┊3888┊  version "1.0.0"
 ┊2417┊3889┊  resolved "https://registry.yarnpkg.com/strip-eof/-/strip-eof-1.0.0.tgz#bb43ff5598a6eb05d89b59fcd129c983313606bf"
```
```diff
@@ -2433,6 +3905,18 @@
 ┊2433┊3905┊    symbol-observable "^1.0.4"
 ┊2434┊3906┊    ws "^5.2.0"
 ┊2435┊3907┊
+┊    ┊3908┊supports-color@^2.0.0:
+┊    ┊3909┊  version "2.0.0"
+┊    ┊3910┊  resolved "https://registry.yarnpkg.com/supports-color/-/supports-color-2.0.0.tgz#535d045ce6b6363fa40117084629995e9df324c7"
+┊    ┊3911┊  integrity sha1-U10EXOa2Nj+kARcIRimZXp3zJMc=
+┊    ┊3912┊
+┊    ┊3913┊supports-color@^4.5.0:
+┊    ┊3914┊  version "4.5.0"
+┊    ┊3915┊  resolved "https://registry.yarnpkg.com/supports-color/-/supports-color-4.5.0.tgz#be7a0de484dec5c5cddf8b3d59125044912f635b"
+┊    ┊3916┊  integrity sha1-vnoN5ITexcXN34s9WRJQRJEvY1s=
+┊    ┊3917┊  dependencies:
+┊    ┊3918┊    has-flag "^2.0.0"
+┊    ┊3919┊
 ┊2436┊3920┊supports-color@^5.2.0, supports-color@^5.3.0:
 ┊2437┊3921┊  version "5.5.0"
 ┊2438┊3922┊  resolved "https://registry.yarnpkg.com/supports-color/-/supports-color-5.5.0.tgz#e2e69a44ac8772f78a1ec0b35b689df6530efc8f"
```
```diff
@@ -2440,7 +3924,15 @@
 ┊2440┊3924┊  dependencies:
 ┊2441┊3925┊    has-flag "^3.0.0"
 ┊2442┊3926┊
-┊2443┊    ┊symbol-observable@^1.0.4:
+┊    ┊3927┊swap-case@^1.1.0:
+┊    ┊3928┊  version "1.1.2"
+┊    ┊3929┊  resolved "https://registry.yarnpkg.com/swap-case/-/swap-case-1.1.2.tgz#c39203a4587385fad3c850a0bd1bcafa081974e3"
+┊    ┊3930┊  integrity sha1-w5IDpFhzhfrTyFCgvRvK+ggZdOM=
+┊    ┊3931┊  dependencies:
+┊    ┊3932┊    lower-case "^1.1.1"
+┊    ┊3933┊    upper-case "^1.1.1"
+┊    ┊3934┊
+┊    ┊3935┊symbol-observable@^1.0.4, symbol-observable@^1.1.0:
 ┊2444┊3936┊  version "1.2.0"
 ┊2445┊3937┊  resolved "https://registry.yarnpkg.com/symbol-observable/-/symbol-observable-1.2.0.tgz#c22688aed4eab3cdc2dfeacbb561660560a00804"
 ┊2446┊3938┊  integrity sha512-e900nM8RRtGhlV36KGEU9k65K3mPb1WV70OdjfxlG2EAuM1noi/E/BaW/uMhL7bPEssK8QV57vN3esixjUvcXQ==
```
```diff
@@ -2465,11 +3957,41 @@
 ┊2465┊3957┊  dependencies:
 ┊2466┊3958┊    execa "^0.7.0"
 ┊2467┊3959┊
+┊    ┊3960┊text-hex@1.0.x:
+┊    ┊3961┊  version "1.0.0"
+┊    ┊3962┊  resolved "https://registry.yarnpkg.com/text-hex/-/text-hex-1.0.0.tgz#69dc9c1b17446ee79a92bf5b884bb4b9127506f5"
+┊    ┊3963┊  integrity sha512-uuVGNWzgJ4yhRaNSiubPY7OjISw4sw4E5Uv0wbjp+OzcbmVU/rsT8ujgcXJhn9ypzsgr5vlzpPqP+MBBKcGvbg==
+┊    ┊3964┊
+┊    ┊3965┊through@^2.3.6:
+┊    ┊3966┊  version "2.3.8"
+┊    ┊3967┊  resolved "https://registry.yarnpkg.com/through/-/through-2.3.8.tgz#0dd4c9ffaabc357960b1b724115d7e0e86a2e1f5"
+┊    ┊3968┊  integrity sha1-DdTJ/6q8NXlgsbckEV1+Doai4fU=
+┊    ┊3969┊
 ┊2468┊3970┊timed-out@^4.0.0:
 ┊2469┊3971┊  version "4.0.1"
 ┊2470┊3972┊  resolved "https://registry.yarnpkg.com/timed-out/-/timed-out-4.0.1.tgz#f32eacac5a175bea25d7fab565ab3ed8741ef56f"
 ┊2471┊3973┊  integrity sha1-8y6srFoXW+ol1/q1Zas+2HQe9W8=
 ┊2472┊3974┊
+┊    ┊3975┊title-case@^2.1.0:
+┊    ┊3976┊  version "2.1.1"
+┊    ┊3977┊  resolved "https://registry.yarnpkg.com/title-case/-/title-case-2.1.1.tgz#3e127216da58d2bc5becf137ab91dae3a7cd8faa"
+┊    ┊3978┊  integrity sha1-PhJyFtpY0rxb7PE3q5Ha46fNj6o=
+┊    ┊3979┊  dependencies:
+┊    ┊3980┊    no-case "^2.2.0"
+┊    ┊3981┊    upper-case "^1.0.3"
+┊    ┊3982┊
+┊    ┊3983┊tmp@^0.0.33:
+┊    ┊3984┊  version "0.0.33"
+┊    ┊3985┊  resolved "https://registry.yarnpkg.com/tmp/-/tmp-0.0.33.tgz#6d34335889768d21b2bcda0aa277ced3b1bfadf9"
+┊    ┊3986┊  integrity sha512-jRCJlojKnZ3addtTOjdIqoRuPEKBvNXcGYqzO6zWZX8KfKEpnGY5jfggJQ3EjKuu8D4bJRr0y+cYJFmYbImXGw==
+┊    ┊3987┊  dependencies:
+┊    ┊3988┊    os-tmpdir "~1.0.2"
+┊    ┊3989┊
+┊    ┊3990┊to-fast-properties@^2.0.0:
+┊    ┊3991┊  version "2.0.0"
+┊    ┊3992┊  resolved "https://registry.yarnpkg.com/to-fast-properties/-/to-fast-properties-2.0.0.tgz#dc5e698cbd079265bc73e0377681a4e4e83f616e"
+┊    ┊3993┊  integrity sha1-3F5pjL0HkmW8c+A3doGk5Og/YW4=
+┊    ┊3994┊
 ┊2473┊3995┊to-object-path@^0.3.0:
 ┊2474┊3996┊  version "0.3.0"
 ┊2475┊3997┊  resolved "https://registry.yarnpkg.com/to-object-path/-/to-object-path-0.3.0.tgz#297588b7b0e7e0ac08e04e672f85c1f4999e17af"
```
```diff
@@ -2507,6 +4029,34 @@
 ┊2507┊4029┊  dependencies:
 ┊2508┊4030┊    nopt "~1.0.10"
 ┊2509┊4031┊
+┊    ┊4032┊tough-cookie@~2.4.3:
+┊    ┊4033┊  version "2.4.3"
+┊    ┊4034┊  resolved "https://registry.yarnpkg.com/tough-cookie/-/tough-cookie-2.4.3.tgz#53f36da3f47783b0925afa06ff9f3b165280f781"
+┊    ┊4035┊  integrity sha512-Q5srk/4vDM54WJsJio3XNn6K2sCG+CQ8G5Wz6bZhRZoAe/+TxjWB/GlFAnYEbkYVlON9FMk/fE3h2RLpPXo4lQ==
+┊    ┊4036┊  dependencies:
+┊    ┊4037┊    psl "^1.1.24"
+┊    ┊4038┊    punycode "^1.4.1"
+┊    ┊4039┊
+┊    ┊4040┊tree-kill@^1.1.0:
+┊    ┊4041┊  version "1.2.1"
+┊    ┊4042┊  resolved "https://registry.yarnpkg.com/tree-kill/-/tree-kill-1.2.1.tgz#5398f374e2f292b9dcc7b2e71e30a5c3bb6c743a"
+┊    ┊4043┊  integrity sha512-4hjqbObwlh2dLyW4tcz0Ymw0ggoaVDMveUB9w8kFSQScdRLo0gxO9J7WFcUBo+W3C1TLdFIEwNOWebgZZ0RH9Q==
+┊    ┊4044┊
+┊    ┊4045┊trim-right@^1.0.1:
+┊    ┊4046┊  version "1.0.1"
+┊    ┊4047┊  resolved "https://registry.yarnpkg.com/trim-right/-/trim-right-1.0.1.tgz#cb2e1203067e0c8de1f614094b9fe45704ea6003"
+┊    ┊4048┊  integrity sha1-yy4SAwZ+DI3h9hQJS5/kVwTqYAM=
+┊    ┊4049┊
+┊    ┊4050┊triple-beam@^1.2.0, triple-beam@^1.3.0:
+┊    ┊4051┊  version "1.3.0"
+┊    ┊4052┊  resolved "https://registry.yarnpkg.com/triple-beam/-/triple-beam-1.3.0.tgz#a595214c7298db8339eeeee083e4d10bd8cb8dd9"
+┊    ┊4053┊  integrity sha512-XrHUvV5HpdLmIj4uVMxHggLbFSZYIn7HEWsqePZcI50pco+MPqJ50wMGY794X7AOOhxOBAjbkqfAbEe/QMp2Lw==
+┊    ┊4054┊
+┊    ┊4055┊ts-log@2.1.4:
+┊    ┊4056┊  version "2.1.4"
+┊    ┊4057┊  resolved "https://registry.yarnpkg.com/ts-log/-/ts-log-2.1.4.tgz#063c5ad1cbab5d49d258d18015963489fb6fb59a"
+┊    ┊4058┊  integrity sha512-P1EJSoyV+N3bR/IWFeAqXzKPZwHpnLY6j7j58mAvewHRipo+BQM2Y1f9Y9BjEQznKwgqqZm7H8iuixmssU7tYQ==
+┊    ┊4059┊
 ┊2510┊4060┊ts-node@^8.0.2:
 ┊2511┊4061┊  version "8.0.2"
 ┊2512┊4062┊  resolved "https://registry.yarnpkg.com/ts-node/-/ts-node-8.0.2.tgz#9ecdf8d782a0ca4c80d1d641cbb236af4ac1b756"
```
```diff
@@ -2518,11 +4068,23 @@
 ┊2518┊4068┊    source-map-support "^0.5.6"
 ┊2519┊4069┊    yn "^3.0.0"
 ┊2520┊4070┊
-┊2521┊    ┊tslib@^1.9.3:
+┊    ┊4071┊tslib@^1.9.0, tslib@^1.9.3:
 ┊2522┊4072┊  version "1.9.3"
 ┊2523┊4073┊  resolved "https://registry.yarnpkg.com/tslib/-/tslib-1.9.3.tgz#d7e4dd79245d85428c4d7e4822a79917954ca286"
 ┊2524┊4074┊  integrity sha512-4krF8scpejhaOgqzBEcGM7yDIEfi0/8+8zDRZhNZZ2kjmHJ4hv3zCbQWxoJGz1iw5U0Jl0nma13xzHXcncMavQ==
 ┊2525┊4075┊
+┊    ┊4076┊tunnel-agent@^0.6.0:
+┊    ┊4077┊  version "0.6.0"
+┊    ┊4078┊  resolved "https://registry.yarnpkg.com/tunnel-agent/-/tunnel-agent-0.6.0.tgz#27a5dea06b36b04a0a9966774b290868f0fc40fd"
+┊    ┊4079┊  integrity sha1-J6XeoGs2sEoKmWZ3SykIaPD8QP0=
+┊    ┊4080┊  dependencies:
+┊    ┊4081┊    safe-buffer "^5.0.1"
+┊    ┊4082┊
+┊    ┊4083┊tweetnacl@^0.14.3, tweetnacl@~0.14.0:
+┊    ┊4084┊  version "0.14.5"
+┊    ┊4085┊  resolved "https://registry.yarnpkg.com/tweetnacl/-/tweetnacl-0.14.5.tgz#5ae68177f192d4456269d108afa93ff8743f4f64"
+┊    ┊4086┊  integrity sha1-WuaBd/GS1EViadEIr6k/+HQ/T2Q=
+┊    ┊4087┊
 ┊2526┊4088┊type-is@^1.6.16, type-is@~1.6.16:
 ┊2527┊4089┊  version "1.6.16"
 ┊2528┊4090┊  resolved "https://registry.yarnpkg.com/type-is/-/type-is-1.6.16.tgz#f89ce341541c672b25ee7ae3c73dee3b2be50194"
```
```diff
@@ -2531,7 +4093,7 @@
 ┊2531┊4093┊    media-typer "0.3.0"
 ┊2532┊4094┊    mime-types "~2.1.18"
 ┊2533┊4095┊
-┊2534┊    ┊typescript@^3.3.3333:
+┊    ┊4096┊typescript@^3.2.2, typescript@^3.3.3333:
 ┊2535┊4097┊  version "3.3.3333"
 ┊2536┊4098┊  resolved "https://registry.yarnpkg.com/typescript/-/typescript-3.3.3333.tgz#171b2c5af66c59e9431199117a3bcadc66fdcfd6"
 ┊2537┊4099┊  integrity sha512-JjSKsAfuHBE/fB2oZ8NxtRTk5iGcg6hkYXMnZ3Wc+b2RSqejEqTaem11mHASMnFilHrax3sLK0GDzcJrekZYLw==
```
```diff
@@ -2599,6 +4161,25 @@
 ┊2599┊4161┊    semver-diff "^2.0.0"
 ┊2600┊4162┊    xdg-basedir "^3.0.0"
 ┊2601┊4163┊
+┊    ┊4164┊upper-case-first@^1.1.0, upper-case-first@^1.1.2:
+┊    ┊4165┊  version "1.1.2"
+┊    ┊4166┊  resolved "https://registry.yarnpkg.com/upper-case-first/-/upper-case-first-1.1.2.tgz#5d79bedcff14419518fd2edb0a0507c9b6859115"
+┊    ┊4167┊  integrity sha1-XXm+3P8UQZUY/S7bCgUHybaFkRU=
+┊    ┊4168┊  dependencies:
+┊    ┊4169┊    upper-case "^1.1.1"
+┊    ┊4170┊
+┊    ┊4171┊upper-case@^1.0.3, upper-case@^1.1.0, upper-case@^1.1.1, upper-case@^1.1.3:
+┊    ┊4172┊  version "1.1.3"
+┊    ┊4173┊  resolved "https://registry.yarnpkg.com/upper-case/-/upper-case-1.1.3.tgz#f6b4501c2ec4cdd26ba78be7222961de77621598"
+┊    ┊4174┊  integrity sha1-9rRQHC7EzdJrp4vnIilh3ndiFZg=
+┊    ┊4175┊
+┊    ┊4176┊uri-js@^4.2.2:
+┊    ┊4177┊  version "4.2.2"
+┊    ┊4178┊  resolved "https://registry.yarnpkg.com/uri-js/-/uri-js-4.2.2.tgz#94c540e1ff772956e2299507c010aea6c8838eb0"
+┊    ┊4179┊  integrity sha512-KY9Frmirql91X2Qgjry0Wd4Y+YTdrdZheS8TFwvkbLWf/G5KNJDCh6pKL5OZctEW4+0Baa5idK2ZQuELRwPznQ==
+┊    ┊4180┊  dependencies:
+┊    ┊4181┊    punycode "^2.1.0"
+┊    ┊4182┊
 ┊2602┊4183┊urix@^0.1.0:
 ┊2603┊4184┊  version "0.1.0"
 ┊2604┊4185┊  resolved "https://registry.yarnpkg.com/urix/-/urix-0.1.0.tgz#da937f7a62e21fec1fd18d49b35c2935067a6c72"
```
```diff
@@ -2616,7 +4197,7 @@
 ┊2616┊4197┊  resolved "https://registry.yarnpkg.com/use/-/use-3.1.1.tgz#d50c8cac79a19fbc20f2911f56eb973f4e10070f"
 ┊2617┊4198┊  integrity sha512-cwESVXlO3url9YWlFW/TA9cshCEhtu7IKJ/p5soJ/gGpj7vbvFrAY/eIioQ6Dw23KjZhYgiIo8HOs1nQ2vr/oQ==
 ┊2618┊4199┊
-┊2619┊    ┊util-deprecate@~1.0.1:
+┊    ┊4200┊util-deprecate@^1.0.1, util-deprecate@~1.0.1:
 ┊2620┊4201┊  version "1.0.2"
 ┊2621┊4202┊  resolved "https://registry.yarnpkg.com/util-deprecate/-/util-deprecate-1.0.2.tgz#450d4dc9fa70de732762fbd2d4a28981419a0ccf"
 ┊2622┊4203┊  integrity sha1-RQ1Nyfpw3nMnYvvS1KKJgUGaDM8=
```
```diff
@@ -2634,16 +4215,48 @@
 ┊2634┊4215┊  resolved "https://registry.yarnpkg.com/utils-merge/-/utils-merge-1.0.1.tgz#9f95710f50a267947b2ccc124741c1028427e713"
 ┊2635┊4216┊  integrity sha1-n5VxD1CiZ5R7LMwSR0HBAoQn5xM=
 ┊2636┊4217┊
-┊2637┊    ┊uuid@^3.1.0:
+┊    ┊4218┊uuid@^3.1.0, uuid@^3.3.2:
 ┊2638┊4219┊  version "3.3.2"
 ┊2639┊4220┊  resolved "https://registry.yarnpkg.com/uuid/-/uuid-3.3.2.tgz#1b4af4955eb3077c501c23872fc6513811587131"
 ┊2640┊4221┊  integrity sha512-yXJmeNaw3DnnKAOKJE51sL/ZaYfWJRl1pK9dr19YFCu0ObS231AB1/LbqTKRAQ5kw8A90rA6fr4riOUpTZvQZA==
 ┊2641┊4222┊
+┊    ┊4223┊valid-url@1.0.9:
+┊    ┊4224┊  version "1.0.9"
+┊    ┊4225┊  resolved "https://registry.yarnpkg.com/valid-url/-/valid-url-1.0.9.tgz#1c14479b40f1397a75782f115e4086447433a200"
+┊    ┊4226┊  integrity sha1-HBRHm0DxOXp1eC8RXkCGRHQzogA=
+┊    ┊4227┊
+┊    ┊4228┊validate-npm-package-license@^3.0.1:
+┊    ┊4229┊  version "3.0.4"
+┊    ┊4230┊  resolved "https://registry.yarnpkg.com/validate-npm-package-license/-/validate-npm-package-license-3.0.4.tgz#fc91f6b9c7ba15c857f4cb2c5defeec39d4f410a"
+┊    ┊4231┊  integrity sha512-DpKm2Ui/xN7/HQKCtpZxoRWBhZ9Z0kqtygG8XCgNQ8ZlDnxuQmWhj566j8fN4Cu3/JmbhsDo7fcAJq4s9h27Ew==
+┊    ┊4232┊  dependencies:
+┊    ┊4233┊    spdx-correct "^3.0.0"
+┊    ┊4234┊    spdx-expression-parse "^3.0.0"
+┊    ┊4235┊
 ┊2642┊4236┊vary@^1, vary@~1.1.2:
 ┊2643┊4237┊  version "1.1.2"
 ┊2644┊4238┊  resolved "https://registry.yarnpkg.com/vary/-/vary-1.1.2.tgz#2299f02c6ded30d4a5961b0b9f74524a18f634fc"
 ┊2645┊4239┊  integrity sha1-IpnwLG3tMNSllhsLn3RSShj2NPw=
 ┊2646┊4240┊
+┊    ┊4241┊verror@1.10.0:
+┊    ┊4242┊  version "1.10.0"
+┊    ┊4243┊  resolved "https://registry.yarnpkg.com/verror/-/verror-1.10.0.tgz#3a105ca17053af55d6e270c1f8288682e18da400"
+┊    ┊4244┊  integrity sha1-OhBcoXBTr1XW4nDB+CiGguGNpAA=
+┊    ┊4245┊  dependencies:
+┊    ┊4246┊    assert-plus "^1.0.0"
+┊    ┊4247┊    core-util-is "1.0.2"
+┊    ┊4248┊    extsprintf "^1.2.0"
+┊    ┊4249┊
+┊    ┊4250┊whatwg-fetch@2.0.4:
+┊    ┊4251┊  version "2.0.4"
+┊    ┊4252┊  resolved "https://registry.yarnpkg.com/whatwg-fetch/-/whatwg-fetch-2.0.4.tgz#dde6a5df315f9d39991aa17621853d720b85566f"
+┊    ┊4253┊  integrity sha512-dcQ1GWpOD/eEQ97k66aiEVpNnapVj90/+R+SXTPYGHpYBBypfKJEQjLrvMZ7YXbKm21gXd4NcuxUTjiv1YtLng==
+┊    ┊4254┊
+┊    ┊4255┊which-module@^2.0.0:
+┊    ┊4256┊  version "2.0.0"
+┊    ┊4257┊  resolved "https://registry.yarnpkg.com/which-module/-/which-module-2.0.0.tgz#d9ef07dce77b9902b8a3a8fa4b31c3e3f7e6e87a"
+┊    ┊4258┊  integrity sha1-2e8H3Od7mQK4o6j6SzHD4/fm6Ho=
+┊    ┊4259┊
 ┊2647┊4260┊which@^1.2.9:
 ┊2648┊4261┊  version "1.3.1"
 ┊2649┊4262┊  resolved "https://registry.yarnpkg.com/which/-/which-1.3.1.tgz#a45043d54f5805316da8d62f9f50918d3da70b0a"
```
```diff
@@ -2665,6 +4278,45 @@
 ┊2665┊4278┊  dependencies:
 ┊2666┊4279┊    string-width "^2.1.1"
 ┊2667┊4280┊
+┊    ┊4281┊winston-transport@^4.3.0:
+┊    ┊4282┊  version "4.3.0"
+┊    ┊4283┊  resolved "https://registry.yarnpkg.com/winston-transport/-/winston-transport-4.3.0.tgz#df68c0c202482c448d9b47313c07304c2d7c2c66"
+┊    ┊4284┊  integrity sha512-B2wPuwUi3vhzn/51Uukcao4dIduEiPOcOt9HJ3QeaXgkJ5Z7UwpBzxS4ZGNHtrxrUvTwemsQiSys0ihOf8Mp1A==
+┊    ┊4285┊  dependencies:
+┊    ┊4286┊    readable-stream "^2.3.6"
+┊    ┊4287┊    triple-beam "^1.2.0"
+┊    ┊4288┊
+┊    ┊4289┊winston@3.2.1:
+┊    ┊4290┊  version "3.2.1"
+┊    ┊4291┊  resolved "https://registry.yarnpkg.com/winston/-/winston-3.2.1.tgz#63061377976c73584028be2490a1846055f77f07"
+┊    ┊4292┊  integrity sha512-zU6vgnS9dAWCEKg/QYigd6cgMVVNwyTzKs81XZtTFuRwJOcDdBg7AU0mXVyNbs7O5RH2zdv+BdNZUlx7mXPuOw==
+┊    ┊4293┊  dependencies:
+┊    ┊4294┊    async "^2.6.1"
+┊    ┊4295┊    diagnostics "^1.1.1"
+┊    ┊4296┊    is-stream "^1.1.0"
+┊    ┊4297┊    logform "^2.1.1"
+┊    ┊4298┊    one-time "0.0.4"
+┊    ┊4299┊    readable-stream "^3.1.1"
+┊    ┊4300┊    stack-trace "0.0.x"
+┊    ┊4301┊    triple-beam "^1.3.0"
+┊    ┊4302┊    winston-transport "^4.3.0"
+┊    ┊4303┊
+┊    ┊4304┊wrap-ansi@^2.0.0:
+┊    ┊4305┊  version "2.1.0"
+┊    ┊4306┊  resolved "https://registry.yarnpkg.com/wrap-ansi/-/wrap-ansi-2.1.0.tgz#d8fc3d284dd05794fe84973caecdd1cf824fdd85"
+┊    ┊4307┊  integrity sha1-2Pw9KE3QV5T+hJc8rs3Rz4JP3YU=
+┊    ┊4308┊  dependencies:
+┊    ┊4309┊    string-width "^1.0.1"
+┊    ┊4310┊    strip-ansi "^3.0.1"
+┊    ┊4311┊
+┊    ┊4312┊wrap-ansi@^3.0.1:
+┊    ┊4313┊  version "3.0.1"
+┊    ┊4314┊  resolved "https://registry.yarnpkg.com/wrap-ansi/-/wrap-ansi-3.0.1.tgz#288a04d87eda5c286e060dfe8f135ce8d007f8ba"
+┊    ┊4315┊  integrity sha1-KIoE2H7aXChuBg3+jxNc6NAH+Lo=
+┊    ┊4316┊  dependencies:
+┊    ┊4317┊    string-width "^2.1.1"
+┊    ┊4318┊    strip-ansi "^4.0.0"
+┊    ┊4319┊
 ┊2668┊4320┊wrappy@1:
 ┊2669┊4321┊  version "1.0.2"
 ┊2670┊4322┊  resolved "https://registry.yarnpkg.com/wrappy/-/wrappy-1.0.2.tgz#b5243d8f3ec1aa35f1364605bc0d1036e30ab69f"
```
```diff
@@ -2698,6 +4350,11 @@
 ┊2698┊4350┊  resolved "https://registry.yarnpkg.com/xdg-basedir/-/xdg-basedir-3.0.0.tgz#496b2cc109eca8dbacfe2dc72b603c17c5870ad4"
 ┊2699┊4351┊  integrity sha1-SWsswQnsqNus/i3HK2A8F8WHCtQ=
 ┊2700┊4352┊
+┊    ┊4353┊"y18n@^3.2.1 || ^4.0.0":
+┊    ┊4354┊  version "4.0.0"
+┊    ┊4355┊  resolved "https://registry.yarnpkg.com/y18n/-/y18n-4.0.0.tgz#95ef94f85ecc81d007c264e190a120f0a3c8566b"
+┊    ┊4356┊  integrity sha512-r9S/ZyXu/Xu9q1tYlpsLIsa3EeLXXk0VwlxqTcFRfg9EhMW+17kbt9G0NrgCmhGb5vT2hyhJZLfDGx+7+5Uj/w==
+┊    ┊4357┊
 ┊2701┊4358┊yallist@^2.1.2:
 ┊2702┊4359┊  version "2.1.2"
 ┊2703┊4360┊  resolved "https://registry.yarnpkg.com/yallist/-/yallist-2.1.2.tgz#1c11f9218f076089a47dd512f93c6699a6a81d52"
```
```diff
@@ -2708,6 +4365,32 @@
 ┊2708┊4365┊  resolved "https://registry.yarnpkg.com/yallist/-/yallist-3.0.3.tgz#b4b049e314be545e3ce802236d6cd22cd91c3de9"
 ┊2709┊4366┊  integrity sha512-S+Zk8DEWE6oKpV+vI3qWkaK+jSbIK86pCwe2IF/xwIpQ8jEuxpw9NyaGjmp9+BoJv5FV2piqCDcoCtStppiq2A==
 ┊2710┊4367┊
+┊    ┊4368┊yargs-parser@^11.1.1:
+┊    ┊4369┊  version "11.1.1"
+┊    ┊4370┊  resolved "https://registry.yarnpkg.com/yargs-parser/-/yargs-parser-11.1.1.tgz#879a0865973bca9f6bab5cbdf3b1c67ec7d3bcf4"
+┊    ┊4371┊  integrity sha512-C6kB/WJDiaxONLJQnF8ccx9SEeoTTLek8RVbaOIsrAUS8VrBEXfmeSnCZxygc+XC2sNMBIwOOnfcxiynjHsVSQ==
+┊    ┊4372┊  dependencies:
+┊    ┊4373┊    camelcase "^5.0.0"
+┊    ┊4374┊    decamelize "^1.2.0"
+┊    ┊4375┊
+┊    ┊4376┊yargs@^12.0.1:
+┊    ┊4377┊  version "12.0.5"
+┊    ┊4378┊  resolved "https://registry.yarnpkg.com/yargs/-/yargs-12.0.5.tgz#05f5997b609647b64f66b81e3b4b10a368e7ad13"
+┊    ┊4379┊  integrity sha512-Lhz8TLaYnxq/2ObqHDql8dX8CJi97oHxrjUcYtzKbbykPtVW9WB+poxI+NM2UIzsMgNCZTIf0AQwsjK5yMAqZw==
+┊    ┊4380┊  dependencies:
+┊    ┊4381┊    cliui "^4.0.0"
+┊    ┊4382┊    decamelize "^1.2.0"
+┊    ┊4383┊    find-up "^3.0.0"
+┊    ┊4384┊    get-caller-file "^1.0.1"
+┊    ┊4385┊    os-locale "^3.0.0"
+┊    ┊4386┊    require-directory "^2.1.1"
+┊    ┊4387┊    require-main-filename "^1.0.1"
+┊    ┊4388┊    set-blocking "^2.0.0"
+┊    ┊4389┊    string-width "^2.0.0"
+┊    ┊4390┊    which-module "^2.0.0"
+┊    ┊4391┊    y18n "^3.2.1 || ^4.0.0"
+┊    ┊4392┊    yargs-parser "^11.1.1"
+┊    ┊4393┊
 ┊2711┊4394┊yn@^3.0.0:
 ┊2712┊4395┊  version "3.0.0"
 ┊2713┊4396┊  resolved "https://registry.yarnpkg.com/yn/-/yn-3.0.0.tgz#0073c6b56e92aed652fbdfd62431f2d6b9a7a091"
```

[}]: #

Now let's run the generator (the server must be running in the background):

    $ npm run generator

Those are the types created with `npm run generator`:

[{]: <helper> (diffStep "2.2")

#### [Step 2.2: Create types with generator](https://github.com/Urigo/WhatsApp-Clone-Server/commit/f31a6a3)

##### Added types.d.ts
```diff
@@ -0,0 +1,507 @@
+┊   ┊  1┊export type Maybe<T> = T | undefined | null;
+┊   ┊  2┊
+┊   ┊  3┊export enum MessageType {
+┊   ┊  4┊  Location = "LOCATION",
+┊   ┊  5┊  Text = "TEXT",
+┊   ┊  6┊  Picture = "PICTURE"
+┊   ┊  7┊}
+┊   ┊  8┊
+┊   ┊  9┊export type Date = any;
+┊   ┊ 10┊
+┊   ┊ 11┊// ====================================================
+┊   ┊ 12┊// Scalars
+┊   ┊ 13┊// ====================================================
+┊   ┊ 14┊
+┊   ┊ 15┊// ====================================================
+┊   ┊ 16┊// Types
+┊   ┊ 17┊// ====================================================
+┊   ┊ 18┊
+┊   ┊ 19┊export interface Query {
+┊   ┊ 20┊  me?: Maybe<User>;
+┊   ┊ 21┊
+┊   ┊ 22┊  users?: Maybe<User[]>;
+┊   ┊ 23┊
+┊   ┊ 24┊  chats: Chat[];
+┊   ┊ 25┊
+┊   ┊ 26┊  chat?: Maybe<Chat>;
+┊   ┊ 27┊}
+┊   ┊ 28┊
+┊   ┊ 29┊export interface User {
+┊   ┊ 30┊  id: string;
+┊   ┊ 31┊
+┊   ┊ 32┊  name?: Maybe<string>;
+┊   ┊ 33┊
+┊   ┊ 34┊  picture?: Maybe<string>;
+┊   ┊ 35┊
+┊   ┊ 36┊  phone?: Maybe<string>;
+┊   ┊ 37┊}
+┊   ┊ 38┊
+┊   ┊ 39┊export interface Chat {
+┊   ┊ 40┊  id: string;
+┊   ┊ 41┊
+┊   ┊ 42┊  createdAt: Date;
+┊   ┊ 43┊
+┊   ┊ 44┊  name?: Maybe<string>;
+┊   ┊ 45┊
+┊   ┊ 46┊  picture?: Maybe<string>;
+┊   ┊ 47┊
+┊   ┊ 48┊  allTimeMembers: User[];
+┊   ┊ 49┊
+┊   ┊ 50┊  listingMembers: User[];
+┊   ┊ 51┊
+┊   ┊ 52┊  actualGroupMembers?: Maybe<User[]>;
+┊   ┊ 53┊
+┊   ┊ 54┊  admins?: Maybe<User[]>;
+┊   ┊ 55┊
+┊   ┊ 56┊  owner?: Maybe<User>;
+┊   ┊ 57┊
+┊   ┊ 58┊  isGroup: boolean;
+┊   ┊ 59┊
+┊   ┊ 60┊  messages: (Maybe<Message>)[];
+┊   ┊ 61┊
+┊   ┊ 62┊  lastMessage?: Maybe<Message>;
+┊   ┊ 63┊
+┊   ┊ 64┊  updatedAt: Date;
+┊   ┊ 65┊
+┊   ┊ 66┊  unreadMessages: number;
+┊   ┊ 67┊}
+┊   ┊ 68┊
+┊   ┊ 69┊export interface Message {
+┊   ┊ 70┊  id: string;
+┊   ┊ 71┊
+┊   ┊ 72┊  sender: User;
+┊   ┊ 73┊
+┊   ┊ 74┊  chat: Chat;
+┊   ┊ 75┊
+┊   ┊ 76┊  content: string;
+┊   ┊ 77┊
+┊   ┊ 78┊  createdAt: Date;
+┊   ┊ 79┊
+┊   ┊ 80┊  type: number;
+┊   ┊ 81┊
+┊   ┊ 82┊  holders: User[];
+┊   ┊ 83┊
+┊   ┊ 84┊  ownership: boolean;
+┊   ┊ 85┊
+┊   ┊ 86┊  recipients: Recipient[];
+┊   ┊ 87┊}
+┊   ┊ 88┊
+┊   ┊ 89┊export interface Recipient {
+┊   ┊ 90┊  user: User;
+┊   ┊ 91┊
+┊   ┊ 92┊  message: Message;
+┊   ┊ 93┊
+┊   ┊ 94┊  chat: Chat;
+┊   ┊ 95┊
+┊   ┊ 96┊  receivedAt?: Maybe<Date>;
+┊   ┊ 97┊
+┊   ┊ 98┊  readAt?: Maybe<Date>;
+┊   ┊ 99┊}
+┊   ┊100┊
+┊   ┊101┊// ====================================================
+┊   ┊102┊// Arguments
+┊   ┊103┊// ====================================================
+┊   ┊104┊
+┊   ┊105┊export interface ChatQueryArgs {
+┊   ┊106┊  chatId: string;
+┊   ┊107┊}
+┊   ┊108┊export interface MessagesChatArgs {
+┊   ┊109┊  amount?: Maybe<number>;
+┊   ┊110┊}
+┊   ┊111┊
+┊   ┊112┊import {
+┊   ┊113┊  GraphQLResolveInfo,
+┊   ┊114┊  GraphQLScalarType,
+┊   ┊115┊  GraphQLScalarTypeConfig
+┊   ┊116┊} from "graphql";
+┊   ┊117┊
+┊   ┊118┊import { ChatDb, MessageDb, RecipientDb, UserDb } from "./db";
+┊   ┊119┊
+┊   ┊120┊export type Resolver<Result, Parent = {}, TContext = {}, Args = {}> = (
+┊   ┊121┊  parent: Parent,
+┊   ┊122┊  args: Args,
+┊   ┊123┊  context: TContext,
+┊   ┊124┊  info: GraphQLResolveInfo
+┊   ┊125┊) => Promise<Result> | Result;
+┊   ┊126┊
+┊   ┊127┊export interface ISubscriptionResolverObject<Result, Parent, TContext, Args> {
+┊   ┊128┊  subscribe<R = Result, P = Parent>(
+┊   ┊129┊    parent: P,
+┊   ┊130┊    args: Args,
+┊   ┊131┊    context: TContext,
+┊   ┊132┊    info: GraphQLResolveInfo
+┊   ┊133┊  ): AsyncIterator<R | Result> | Promise<AsyncIterator<R | Result>>;
+┊   ┊134┊  resolve?<R = Result, P = Parent>(
+┊   ┊135┊    parent: P,
+┊   ┊136┊    args: Args,
+┊   ┊137┊    context: TContext,
+┊   ┊138┊    info: GraphQLResolveInfo
+┊   ┊139┊  ): R | Result | Promise<R | Result>;
+┊   ┊140┊}
+┊   ┊141┊
+┊   ┊142┊export type SubscriptionResolver<
+┊   ┊143┊  Result,
+┊   ┊144┊  Parent = {},
+┊   ┊145┊  TContext = {},
+┊   ┊146┊  Args = {}
+┊   ┊147┊> =
+┊   ┊148┊  | ((
+┊   ┊149┊      ...args: any[]
+┊   ┊150┊    ) => ISubscriptionResolverObject<Result, Parent, TContext, Args>)
+┊   ┊151┊  | ISubscriptionResolverObject<Result, Parent, TContext, Args>;
+┊   ┊152┊
+┊   ┊153┊export type TypeResolveFn<Types, Parent = {}, TContext = {}> = (
+┊   ┊154┊  parent: Parent,
+┊   ┊155┊  context: TContext,
+┊   ┊156┊  info: GraphQLResolveInfo
+┊   ┊157┊) => Maybe<Types>;
+┊   ┊158┊
+┊   ┊159┊export type NextResolverFn<T> = () => Promise<T>;
+┊   ┊160┊
+┊   ┊161┊export type DirectiveResolverFn<TResult, TArgs = {}, TContext = {}> = (
+┊   ┊162┊  next: NextResolverFn<TResult>,
+┊   ┊163┊  source: any,
+┊   ┊164┊  args: TArgs,
+┊   ┊165┊  context: TContext,
+┊   ┊166┊  info: GraphQLResolveInfo
+┊   ┊167┊) => TResult | Promise<TResult>;
+┊   ┊168┊
+┊   ┊169┊export namespace QueryResolvers {
+┊   ┊170┊  export interface Resolvers<TContext = {}, TypeParent = {}> {
+┊   ┊171┊    me?: MeResolver<Maybe<UserDb>, TypeParent, TContext>;
+┊   ┊172┊
+┊   ┊173┊    users?: UsersResolver<Maybe<UserDb[]>, TypeParent, TContext>;
+┊   ┊174┊
+┊   ┊175┊    chats?: ChatsResolver<ChatDb[], TypeParent, TContext>;
+┊   ┊176┊
+┊   ┊177┊    chat?: ChatResolver<Maybe<ChatDb>, TypeParent, TContext>;
+┊   ┊178┊  }
+┊   ┊179┊
+┊   ┊180┊  export type MeResolver<
+┊   ┊181┊    R = Maybe<UserDb>,
+┊   ┊182┊    Parent = {},
+┊   ┊183┊    TContext = {}
+┊   ┊184┊  > = Resolver<R, Parent, TContext>;
+┊   ┊185┊  export type UsersResolver<
+┊   ┊186┊    R = Maybe<UserDb[]>,
+┊   ┊187┊    Parent = {},
+┊   ┊188┊    TContext = {}
+┊   ┊189┊  > = Resolver<R, Parent, TContext>;
+┊   ┊190┊  export type ChatsResolver<
+┊   ┊191┊    R = ChatDb[],
+┊   ┊192┊    Parent = {},
+┊   ┊193┊    TContext = {}
+┊   ┊194┊  > = Resolver<R, Parent, TContext>;
+┊   ┊195┊  export type ChatResolver<
+┊   ┊196┊    R = Maybe<ChatDb>,
+┊   ┊197┊    Parent = {},
+┊   ┊198┊    TContext = {}
+┊   ┊199┊  > = Resolver<R, Parent, TContext, ChatArgs>;
+┊   ┊200┊  export interface ChatArgs {
+┊   ┊201┊    chatId: string;
+┊   ┊202┊  }
+┊   ┊203┊}
+┊   ┊204┊
+┊   ┊205┊export namespace UserResolvers {
+┊   ┊206┊  export interface Resolvers<TContext = {}, TypeParent = UserDb> {
+┊   ┊207┊    id?: IdResolver<string, TypeParent, TContext>;
+┊   ┊208┊
+┊   ┊209┊    name?: NameResolver<Maybe<string>, TypeParent, TContext>;
+┊   ┊210┊
+┊   ┊211┊    picture?: PictureResolver<Maybe<string>, TypeParent, TContext>;
+┊   ┊212┊
+┊   ┊213┊    phone?: PhoneResolver<Maybe<string>, TypeParent, TContext>;
+┊   ┊214┊  }
+┊   ┊215┊
+┊   ┊216┊  export type IdResolver<R = string, Parent = UserDb, TContext = {}> = Resolver<
+┊   ┊217┊    R,
+┊   ┊218┊    Parent,
+┊   ┊219┊    TContext
+┊   ┊220┊  >;
+┊   ┊221┊  export type NameResolver<
+┊   ┊222┊    R = Maybe<string>,
+┊   ┊223┊    Parent = UserDb,
+┊   ┊224┊    TContext = {}
+┊   ┊225┊  > = Resolver<R, Parent, TContext>;
+┊   ┊226┊  export type PictureResolver<
+┊   ┊227┊    R = Maybe<string>,
+┊   ┊228┊    Parent = UserDb,
+┊   ┊229┊    TContext = {}
+┊   ┊230┊  > = Resolver<R, Parent, TContext>;
+┊   ┊231┊  export type PhoneResolver<
+┊   ┊232┊    R = Maybe<string>,
+┊   ┊233┊    Parent = UserDb,
+┊   ┊234┊    TContext = {}
+┊   ┊235┊  > = Resolver<R, Parent, TContext>;
+┊   ┊236┊}
+┊   ┊237┊
+┊   ┊238┊export namespace ChatResolvers {
+┊   ┊239┊  export interface Resolvers<TContext = {}, TypeParent = ChatDb> {
+┊   ┊240┊    id?: IdResolver<string, TypeParent, TContext>;
+┊   ┊241┊
+┊   ┊242┊    createdAt?: CreatedAtResolver<Date, TypeParent, TContext>;
+┊   ┊243┊
+┊   ┊244┊    name?: NameResolver<Maybe<string>, TypeParent, TContext>;
+┊   ┊245┊
+┊   ┊246┊    picture?: PictureResolver<Maybe<string>, TypeParent, TContext>;
+┊   ┊247┊
+┊   ┊248┊    allTimeMembers?: AllTimeMembersResolver<UserDb[], TypeParent, TContext>;
+┊   ┊249┊
+┊   ┊250┊    listingMembers?: ListingMembersResolver<UserDb[], TypeParent, TContext>;
+┊   ┊251┊
+┊   ┊252┊    actualGroupMembers?: ActualGroupMembersResolver<
+┊   ┊253┊      Maybe<UserDb[]>,
+┊   ┊254┊      TypeParent,
+┊   ┊255┊      TContext
+┊   ┊256┊    >;
+┊   ┊257┊
+┊   ┊258┊    admins?: AdminsResolver<Maybe<UserDb[]>, TypeParent, TContext>;
+┊   ┊259┊
+┊   ┊260┊    owner?: OwnerResolver<Maybe<UserDb>, TypeParent, TContext>;
+┊   ┊261┊
+┊   ┊262┊    isGroup?: IsGroupResolver<boolean, TypeParent, TContext>;
+┊   ┊263┊
+┊   ┊264┊    messages?: MessagesResolver<(Maybe<MessageDb>)[], TypeParent, TContext>;
+┊   ┊265┊
+┊   ┊266┊    lastMessage?: LastMessageResolver<Maybe<MessageDb>, TypeParent, TContext>;
+┊   ┊267┊
+┊   ┊268┊    updatedAt?: UpdatedAtResolver<Date, TypeParent, TContext>;
+┊   ┊269┊
+┊   ┊270┊    unreadMessages?: UnreadMessagesResolver<number, TypeParent, TContext>;
+┊   ┊271┊  }
+┊   ┊272┊
+┊   ┊273┊  export type IdResolver<R = string, Parent = ChatDb, TContext = {}> = Resolver<
+┊   ┊274┊    R,
+┊   ┊275┊    Parent,
+┊   ┊276┊    TContext
+┊   ┊277┊  >;
+┊   ┊278┊  export type CreatedAtResolver<
+┊   ┊279┊    R = Date,
+┊   ┊280┊    Parent = ChatDb,
+┊   ┊281┊    TContext = {}
+┊   ┊282┊  > = Resolver<R, Parent, TContext>;
+┊   ┊283┊  export type NameResolver<
+┊   ┊284┊    R = Maybe<string>,
+┊   ┊285┊    Parent = ChatDb,
+┊   ┊286┊    TContext = {}
+┊   ┊287┊  > = Resolver<R, Parent, TContext>;
+┊   ┊288┊  export type PictureResolver<
+┊   ┊289┊    R = Maybe<string>,
+┊   ┊290┊    Parent = ChatDb,
+┊   ┊291┊    TContext = {}
+┊   ┊292┊  > = Resolver<R, Parent, TContext>;
+┊   ┊293┊  export type AllTimeMembersResolver<
+┊   ┊294┊    R = UserDb[],
+┊   ┊295┊    Parent = ChatDb,
+┊   ┊296┊    TContext = {}
+┊   ┊297┊  > = Resolver<R, Parent, TContext>;
+┊   ┊298┊  export type ListingMembersResolver<
+┊   ┊299┊    R = UserDb[],
+┊   ┊300┊    Parent = ChatDb,
+┊   ┊301┊    TContext = {}
+┊   ┊302┊  > = Resolver<R, Parent, TContext>;
+┊   ┊303┊  export type ActualGroupMembersResolver<
+┊   ┊304┊    R = Maybe<UserDb[]>,
+┊   ┊305┊    Parent = ChatDb,
+┊   ┊306┊    TContext = {}
+┊   ┊307┊  > = Resolver<R, Parent, TContext>;
+┊   ┊308┊  export type AdminsResolver<
+┊   ┊309┊    R = Maybe<UserDb[]>,
+┊   ┊310┊    Parent = ChatDb,
+┊   ┊311┊    TContext = {}
+┊   ┊312┊  > = Resolver<R, Parent, TContext>;
+┊   ┊313┊  export type OwnerResolver<
+┊   ┊314┊    R = Maybe<UserDb>,
+┊   ┊315┊    Parent = ChatDb,
+┊   ┊316┊    TContext = {}
+┊   ┊317┊  > = Resolver<R, Parent, TContext>;
+┊   ┊318┊  export type IsGroupResolver<
+┊   ┊319┊    R = boolean,
+┊   ┊320┊    Parent = ChatDb,
+┊   ┊321┊    TContext = {}
+┊   ┊322┊  > = Resolver<R, Parent, TContext>;
+┊   ┊323┊  export type MessagesResolver<
+┊   ┊324┊    R = (Maybe<MessageDb>)[],
+┊   ┊325┊    Parent = ChatDb,
+┊   ┊326┊    TContext = {}
+┊   ┊327┊  > = Resolver<R, Parent, TContext, MessagesArgs>;
+┊   ┊328┊  export interface MessagesArgs {
+┊   ┊329┊    amount?: Maybe<number>;
+┊   ┊330┊  }
+┊   ┊331┊
+┊   ┊332┊  export type LastMessageResolver<
+┊   ┊333┊    R = Maybe<MessageDb>,
+┊   ┊334┊    Parent = ChatDb,
+┊   ┊335┊    TContext = {}
+┊   ┊336┊  > = Resolver<R, Parent, TContext>;
+┊   ┊337┊  export type UpdatedAtResolver<
+┊   ┊338┊    R = Date,
+┊   ┊339┊    Parent = ChatDb,
+┊   ┊340┊    TContext = {}
+┊   ┊341┊  > = Resolver<R, Parent, TContext>;
+┊   ┊342┊  export type UnreadMessagesResolver<
+┊   ┊343┊    R = number,
+┊   ┊344┊    Parent = ChatDb,
+┊   ┊345┊    TContext = {}
+┊   ┊346┊  > = Resolver<R, Parent, TContext>;
+┊   ┊347┊}
+┊   ┊348┊
+┊   ┊349┊export namespace MessageResolvers {
+┊   ┊350┊  export interface Resolvers<TContext = {}, TypeParent = MessageDb> {
+┊   ┊351┊    id?: IdResolver<string, TypeParent, TContext>;
+┊   ┊352┊
+┊   ┊353┊    sender?: SenderResolver<UserDb, TypeParent, TContext>;
+┊   ┊354┊
+┊   ┊355┊    chat?: ChatResolver<ChatDb, TypeParent, TContext>;
+┊   ┊356┊
+┊   ┊357┊    content?: ContentResolver<string, TypeParent, TContext>;
+┊   ┊358┊
+┊   ┊359┊    createdAt?: CreatedAtResolver<Date, TypeParent, TContext>;
+┊   ┊360┊
+┊   ┊361┊    type?: TypeResolver<number, TypeParent, TContext>;
+┊   ┊362┊
+┊   ┊363┊    holders?: HoldersResolver<UserDb[], TypeParent, TContext>;
+┊   ┊364┊
+┊   ┊365┊    ownership?: OwnershipResolver<boolean, TypeParent, TContext>;
+┊   ┊366┊
+┊   ┊367┊    recipients?: RecipientsResolver<RecipientDb[], TypeParent, TContext>;
+┊   ┊368┊  }
+┊   ┊369┊
+┊   ┊370┊  export type IdResolver<
+┊   ┊371┊    R = string,
+┊   ┊372┊    Parent = MessageDb,
+┊   ┊373┊    TContext = {}
+┊   ┊374┊  > = Resolver<R, Parent, TContext>;
+┊   ┊375┊  export type SenderResolver<
+┊   ┊376┊    R = UserDb,
+┊   ┊377┊    Parent = MessageDb,
+┊   ┊378┊    TContext = {}
+┊   ┊379┊  > = Resolver<R, Parent, TContext>;
+┊   ┊380┊  export type ChatResolver<
+┊   ┊381┊    R = ChatDb,
+┊   ┊382┊    Parent = MessageDb,
+┊   ┊383┊    TContext = {}
+┊   ┊384┊  > = Resolver<R, Parent, TContext>;
+┊   ┊385┊  export type ContentResolver<
+┊   ┊386┊    R = string,
+┊   ┊387┊    Parent = MessageDb,
+┊   ┊388┊    TContext = {}
+┊   ┊389┊  > = Resolver<R, Parent, TContext>;
+┊   ┊390┊  export type CreatedAtResolver<
+┊   ┊391┊    R = Date,
+┊   ┊392┊    Parent = MessageDb,
+┊   ┊393┊    TContext = {}
+┊   ┊394┊  > = Resolver<R, Parent, TContext>;
+┊   ┊395┊  export type TypeResolver<
+┊   ┊396┊    R = number,
+┊   ┊397┊    Parent = MessageDb,
+┊   ┊398┊    TContext = {}
+┊   ┊399┊  > = Resolver<R, Parent, TContext>;
+┊   ┊400┊  export type HoldersResolver<
+┊   ┊401┊    R = UserDb[],
+┊   ┊402┊    Parent = MessageDb,
+┊   ┊403┊    TContext = {}
+┊   ┊404┊  > = Resolver<R, Parent, TContext>;
+┊   ┊405┊  export type OwnershipResolver<
+┊   ┊406┊    R = boolean,
+┊   ┊407┊    Parent = MessageDb,
+┊   ┊408┊    TContext = {}
+┊   ┊409┊  > = Resolver<R, Parent, TContext>;
+┊   ┊410┊  export type RecipientsResolver<
+┊   ┊411┊    R = RecipientDb[],
+┊   ┊412┊    Parent = MessageDb,
+┊   ┊413┊    TContext = {}
+┊   ┊414┊  > = Resolver<R, Parent, TContext>;
+┊   ┊415┊}
+┊   ┊416┊
+┊   ┊417┊export namespace RecipientResolvers {
+┊   ┊418┊  export interface Resolvers<TContext = {}, TypeParent = RecipientDb> {
+┊   ┊419┊    user?: UserResolver<UserDb, TypeParent, TContext>;
+┊   ┊420┊
+┊   ┊421┊    message?: MessageResolver<MessageDb, TypeParent, TContext>;
+┊   ┊422┊
+┊   ┊423┊    chat?: ChatResolver<ChatDb, TypeParent, TContext>;
+┊   ┊424┊
+┊   ┊425┊    receivedAt?: ReceivedAtResolver<Maybe<Date>, TypeParent, TContext>;
+┊   ┊426┊
+┊   ┊427┊    readAt?: ReadAtResolver<Maybe<Date>, TypeParent, TContext>;
+┊   ┊428┊  }
+┊   ┊429┊
+┊   ┊430┊  export type UserResolver<
+┊   ┊431┊    R = UserDb,
+┊   ┊432┊    Parent = RecipientDb,
+┊   ┊433┊    TContext = {}
+┊   ┊434┊  > = Resolver<R, Parent, TContext>;
+┊   ┊435┊  export type MessageResolver<
+┊   ┊436┊    R = MessageDb,
+┊   ┊437┊    Parent = RecipientDb,
+┊   ┊438┊    TContext = {}
+┊   ┊439┊  > = Resolver<R, Parent, TContext>;
+┊   ┊440┊  export type ChatResolver<
+┊   ┊441┊    R = ChatDb,
+┊   ┊442┊    Parent = RecipientDb,
+┊   ┊443┊    TContext = {}
+┊   ┊444┊  > = Resolver<R, Parent, TContext>;
+┊   ┊445┊  export type ReceivedAtResolver<
+┊   ┊446┊    R = Maybe<Date>,
+┊   ┊447┊    Parent = RecipientDb,
+┊   ┊448┊    TContext = {}
+┊   ┊449┊  > = Resolver<R, Parent, TContext>;
+┊   ┊450┊  export type ReadAtResolver<
+┊   ┊451┊    R = Maybe<Date>,
+┊   ┊452┊    Parent = RecipientDb,
+┊   ┊453┊    TContext = {}
+┊   ┊454┊  > = Resolver<R, Parent, TContext>;
+┊   ┊455┊}
+┊   ┊456┊
+┊   ┊457┊/** Directs the executor to skip this field or fragment when the `if` argument is true. */
+┊   ┊458┊export type SkipDirectiveResolver<Result> = DirectiveResolverFn<
+┊   ┊459┊  Result,
+┊   ┊460┊  SkipDirectiveArgs,
+┊   ┊461┊  {}
+┊   ┊462┊>;
+┊   ┊463┊export interface SkipDirectiveArgs {
+┊   ┊464┊  /** Skipped when true. */
+┊   ┊465┊  if: boolean;
+┊   ┊466┊}
+┊   ┊467┊
+┊   ┊468┊/** Directs the executor to include this field or fragment only when the `if` argument is true. */
+┊   ┊469┊export type IncludeDirectiveResolver<Result> = DirectiveResolverFn<
+┊   ┊470┊  Result,
+┊   ┊471┊  IncludeDirectiveArgs,
+┊   ┊472┊  {}
+┊   ┊473┊>;
+┊   ┊474┊export interface IncludeDirectiveArgs {
+┊   ┊475┊  /** Included when true. */
+┊   ┊476┊  if: boolean;
+┊   ┊477┊}
+┊   ┊478┊
+┊   ┊479┊/** Marks an element of a GraphQL schema as no longer supported. */
+┊   ┊480┊export type DeprecatedDirectiveResolver<Result> = DirectiveResolverFn<
+┊   ┊481┊  Result,
+┊   ┊482┊  DeprecatedDirectiveArgs,
+┊   ┊483┊  {}
+┊   ┊484┊>;
+┊   ┊485┊export interface DeprecatedDirectiveArgs {
+┊   ┊486┊  /** Explains why this element was deprecated, usually also including a suggestion for how to access supported similar data. Formatted using the Markdown syntax (as specified by [CommonMark](https://commonmark.org/). */
+┊   ┊487┊  reason?: string;
+┊   ┊488┊}
+┊   ┊489┊
+┊   ┊490┊export interface DateScalarConfig extends GraphQLScalarTypeConfig<Date, any> {
+┊   ┊491┊  name: "Date";
+┊   ┊492┊}
+┊   ┊493┊
+┊   ┊494┊export interface IResolvers<TContext = {}> {
+┊   ┊495┊  Query?: QueryResolvers.Resolvers<TContext>;
+┊   ┊496┊  User?: UserResolvers.Resolvers<TContext>;
+┊   ┊497┊  Chat?: ChatResolvers.Resolvers<TContext>;
+┊   ┊498┊  Message?: MessageResolvers.Resolvers<TContext>;
+┊   ┊499┊  Recipient?: RecipientResolvers.Resolvers<TContext>;
+┊   ┊500┊  Date?: GraphQLScalarType;
+┊   ┊501┊}
+┊   ┊502┊
+┊   ┊503┊export interface IDirectiveResolvers<Result> {
+┊   ┊504┊  skip?: SkipDirectiveResolver<Result>;
+┊   ┊505┊  include?: IncludeDirectiveResolver<Result>;
+┊   ┊506┊  deprecated?: DeprecatedDirectiveResolver<Result>;
+┊   ┊507┊}
```

[}]: #

Now let's use them:

[{]: <helper> (diffStep "2.3")

#### [Step 2.3: Use our types](https://github.com/Urigo/WhatsApp-Clone-Server/commit/295d98b)

##### Changed schema&#x2F;index.ts
```diff
@@ -4,5 +4,5 @@
 ┊4┊4┊
 ┊5┊5┊export const schema = makeExecutableSchema({
 ┊6┊6┊  typeDefs,
-┊7┊ ┊  resolvers,
+┊ ┊7┊  resolvers: resolvers as any,
 ┊8┊8┊});
```

##### Changed schema&#x2F;resolvers.ts
```diff
@@ -1,6 +1,6 @@
-┊1┊ ┊import { IResolvers } from 'apollo-server-express';
+┊ ┊1┊import { db } from "../db";
 ┊2┊2┊import { GraphQLDateTime } from 'graphql-iso-date';
-┊3┊ ┊import { ChatDb, db, MessageDb, RecipientDb, UserDb } from "../db";
+┊ ┊3┊import { IResolvers } from '../types';
 ┊4┊4┊
 ┊5┊5┊let users = db.users;
 ┊6┊6┊let chats = db.chats;
```
```diff
@@ -9,57 +9,57 @@
 ┊ 9┊ 9┊export const resolvers: IResolvers = {
 ┊10┊10┊  Date: GraphQLDateTime,
 ┊11┊11┊  Query: {
-┊12┊  ┊    me: (): UserDb => currentUser,
-┊13┊  ┊    users: (): UserDb[] => users.filter(user => user.id !== currentUser.id),
-┊14┊  ┊    chats: (): ChatDb[] => chats.filter(chat => chat.listingMemberIds.includes(currentUser.id)),
-┊15┊  ┊    chat: (obj: any, {chatId}): ChatDb | null => chats.find(chat => chat.id === chatId) || null,
+┊  ┊12┊    me: () => currentUser,
+┊  ┊13┊    users: () => users.filter(user => user.id !== currentUser.id),
+┊  ┊14┊    chats: () => chats.filter(chat => chat.listingMemberIds.includes(currentUser.id)),
+┊  ┊15┊    chat: (obj, {chatId}) => chats.find(chat => chat.id === Number(chatId)),
 ┊16┊16┊  },
 ┊17┊17┊  Chat: {
-┊18┊  ┊    name: (chat: ChatDb): string => chat.name ? chat.name : users
+┊  ┊18┊    name: (chat) => chat.name ? chat.name : users
 ┊19┊19┊      .find(user => user.id === chat.allTimeMemberIds.find(userId => userId !== currentUser.id))!.name,
-┊20┊  ┊    picture: (chat: ChatDb) => chat.name ? chat.picture : users
+┊  ┊20┊    picture: (chat) => chat.name ? chat.picture : users
 ┊21┊21┊      .find(user => user.id === chat.allTimeMemberIds.find(userId => userId !== currentUser.id))!.picture,
-┊22┊  ┊    allTimeMembers: (chat: ChatDb): UserDb[] => users.filter(user => chat.allTimeMemberIds.includes(user.id)),
-┊23┊  ┊    listingMembers: (chat: ChatDb): UserDb[] => users.filter(user => chat.listingMemberIds.includes(user.id)),
-┊24┊  ┊    actualGroupMembers: (chat: ChatDb): UserDb[] => users.filter(user => chat.actualGroupMemberIds && chat.actualGroupMemberIds.includes(user.id)),
-┊25┊  ┊    admins: (chat: ChatDb): UserDb[] => users.filter(user => chat.adminIds && chat.adminIds.includes(user.id)),
-┊26┊  ┊    owner: (chat: ChatDb): UserDb | null => users.find(user => chat.ownerId === user.id) || null,
-┊27┊  ┊    isGroup: (chat: ChatDb): boolean => !!chat.name,
-┊28┊  ┊    messages: (chat: ChatDb, {amount = 0}: {amount: number}): MessageDb[] => {
+┊  ┊22┊    allTimeMembers: (chat) => users.filter(user => chat.allTimeMemberIds.includes(user.id)),
+┊  ┊23┊    listingMembers: (chat) => users.filter(user => chat.listingMemberIds.includes(user.id)),
+┊  ┊24┊    actualGroupMembers: (chat) => users.filter(user => chat.actualGroupMemberIds && chat.actualGroupMemberIds.includes(user.id)),
+┊  ┊25┊    admins: (chat) => users.filter(user => chat.adminIds && chat.adminIds.includes(user.id)),
+┊  ┊26┊    owner: (chat) => users.find(user => chat.ownerId === user.id) || null,
+┊  ┊27┊    isGroup: (chat) => !!chat.name,
+┊  ┊28┊    messages: (chat, {amount = 0}) => {
 ┊29┊29┊      const messages = chat.messages
 ┊30┊30┊      .filter(message => message.holderIds.includes(currentUser.id))
-┊31┊  ┊      .sort((a, b) => b.createdAt.valueOf() - a.createdAt.valueOf()) || <MessageDb[]>[];
+┊  ┊31┊      .sort((a, b) => b.createdAt.valueOf() - a.createdAt.valueOf()) || [];
 ┊32┊32┊      return (amount ? messages.slice(0, amount) : messages).reverse();
 ┊33┊33┊    },
-┊34┊  ┊    lastMessage: (chat: ChatDb): MessageDb => {
+┊  ┊34┊    lastMessage: (chat) => {
 ┊35┊35┊      return chat.messages
 ┊36┊36┊        .filter(message => message.holderIds.includes(currentUser.id))
 ┊37┊37┊        .sort((a, b) => b.createdAt.valueOf() - a.createdAt.valueOf())[0] || null;
 ┊38┊38┊    },
-┊39┊  ┊    updatedAt: (chat: ChatDb): Date => {
+┊  ┊39┊    updatedAt: (chat) => {
 ┊40┊40┊      const lastMessage = chat.messages
 ┊41┊41┊        .filter(message => message.holderIds.includes(currentUser.id))
 ┊42┊42┊        .sort((a, b) => b.createdAt.valueOf() - a.createdAt.valueOf())[0];
 ┊43┊43┊
 ┊44┊44┊      return lastMessage ? lastMessage.createdAt : chat.createdAt;
 ┊45┊45┊    },
-┊46┊  ┊    unreadMessages: (chat: ChatDb): number => chat.messages
+┊  ┊46┊    unreadMessages: (chat) => chat.messages
 ┊47┊47┊      .filter(message => message.holderIds.includes(currentUser.id) &&
 ┊48┊48┊        message.recipients.find(recipient => recipient.userId === currentUser.id && !recipient.readAt))
 ┊49┊49┊      .length,
 ┊50┊50┊  },
 ┊51┊51┊  Message: {
-┊52┊  ┊    chat: (message: MessageDb): ChatDb | null => chats.find(chat => message.chatId === chat.id) || null,
-┊53┊  ┊    sender: (message: MessageDb): UserDb | null => users.find(user => user.id === message.senderId) || null,
-┊54┊  ┊    holders: (message: MessageDb): UserDb[] => users.filter(user => message.holderIds.includes(user.id)),
-┊55┊  ┊    ownership: (message: MessageDb): boolean => message.senderId === currentUser.id,
+┊  ┊52┊    chat: (message) => chats.find(chat => message.chatId === chat.id)!,
+┊  ┊53┊    sender: (message) => users.find(user => user.id === message.senderId)!,
+┊  ┊54┊    holders: (message) => users.filter(user => message.holderIds.includes(user.id)),
+┊  ┊55┊    ownership: (message) => message.senderId === currentUser.id,
 ┊56┊56┊  },
 ┊57┊57┊  Recipient: {
-┊58┊  ┊    user: (recipient: RecipientDb): UserDb | null => users.find(user => recipient.userId === user.id) || null,
-┊59┊  ┊    message: (recipient: RecipientDb): MessageDb | null => {
-┊60┊  ┊      const chat = chats.find(chat => recipient.chatId === chat.id);
-┊61┊  ┊      return chat ? chat.messages.find(message => recipient.messageId === message.id) || null : null;
+┊  ┊58┊    user: (recipient) => users.find(user => recipient.userId === user.id)!,
+┊  ┊59┊    message: (recipient) => {
+┊  ┊60┊      const chat = chats.find(chat => recipient.chatId === chat.id)!;
+┊  ┊61┊      return chat.messages.find(message => recipient.messageId === message.id)!;
 ┊62┊62┊    },
-┊63┊  ┊    chat: (recipient: RecipientDb): ChatDb | null => chats.find(chat => recipient.chatId === chat.id) || null,
+┊  ┊63┊    chat: (recipient) => chats.find(chat => recipient.chatId === chat.id)!,
 ┊64┊64┊  },
 ┊65┊65┊};
```

[}]: #

Don't worry, they will be much more useful when we will write our first mutation.

# Chapter 9

Finally we're going to create our mutations in the server:

[{]: <helper> (diffStep "3.1")

#### [Step 3.1: Add mutations](https://github.com/Urigo/WhatsApp-Clone-Server/commit/9953480)

##### Changed schema&#x2F;resolvers.ts
```diff
@@ -1,6 +1,7 @@
-┊1┊ ┊import { db } from "../db";
+┊ ┊1┊import { IResolvers } from "../types";
 ┊2┊2┊import { GraphQLDateTime } from 'graphql-iso-date';
-┊3┊ ┊import { IResolvers } from '../types';
+┊ ┊3┊import { ChatDb, db, MessageDb, MessageType, RecipientDb } from "../db";
+┊ ┊4┊import moment from "moment";
 ┊4┊5┊
 ┊5┊6┊let users = db.users;
 ┊6┊7┊let chats = db.chats;
```
```diff
@@ -14,6 +15,298 @@
 ┊ 14┊ 15┊    chats: () => chats.filter(chat => chat.listingMemberIds.includes(currentUser.id)),
 ┊ 15┊ 16┊    chat: (obj, {chatId}) => chats.find(chat => chat.id === Number(chatId)),
 ┊ 16┊ 17┊  },
+┊   ┊ 18┊  Mutation: {
+┊   ┊ 19┊    updateUser: (obj, {name, picture}) => {
+┊   ┊ 20┊      currentUser.name = name || currentUser.name;
+┊   ┊ 21┊      currentUser.picture = picture || currentUser.picture;
+┊   ┊ 22┊
+┊   ┊ 23┊      return currentUser;
+┊   ┊ 24┊    },
+┊   ┊ 25┊    addChat: (obj, {userId}) => {
+┊   ┊ 26┊      if (!users.find(user => user.id === Number(userId))) {
+┊   ┊ 27┊        throw new Error(`User ${userId} doesn't exist.`);
+┊   ┊ 28┊      }
+┊   ┊ 29┊
+┊   ┊ 30┊      const chat = chats.find(chat => !chat.name && chat.allTimeMemberIds.includes(currentUser.id) && chat.allTimeMemberIds.includes(Number(userId)));
+┊   ┊ 31┊      if (chat) {
+┊   ┊ 32┊        // Chat already exists. Both users are already in the allTimeMemberIds array
+┊   ┊ 33┊        const chatId = chat.id;
+┊   ┊ 34┊        if (!chat.listingMemberIds.includes(currentUser.id)) {
+┊   ┊ 35┊          // The chat isn't listed for the current user. Add him to the memberIds
+┊   ┊ 36┊          chat.listingMemberIds.push(currentUser.id);
+┊   ┊ 37┊          chats.find(chat => chat.id === chatId)!.listingMemberIds.push(currentUser.id);
+┊   ┊ 38┊          return chat;
+┊   ┊ 39┊        } else {
+┊   ┊ 40┊          throw new Error(`Chat already exists.`);
+┊   ┊ 41┊        }
+┊   ┊ 42┊      } else {
+┊   ┊ 43┊        // Create the chat
+┊   ┊ 44┊        const id = (chats.length && chats[chats.length - 1].id + 1) || 1;
+┊   ┊ 45┊        const chat: ChatDb = {
+┊   ┊ 46┊          id,
+┊   ┊ 47┊          createdAt: moment().toDate(),
+┊   ┊ 48┊          name: null,
+┊   ┊ 49┊          picture: null,
+┊   ┊ 50┊          adminIds: null,
+┊   ┊ 51┊          ownerId: null,
+┊   ┊ 52┊          allTimeMemberIds: [currentUser.id, Number(userId)],
+┊   ┊ 53┊          // Chat will not be listed to the other user until the first message gets written
+┊   ┊ 54┊          listingMemberIds: [currentUser.id],
+┊   ┊ 55┊          actualGroupMemberIds: null,
+┊   ┊ 56┊          messages: [],
+┊   ┊ 57┊        };
+┊   ┊ 58┊        chats.push(chat);
+┊   ┊ 59┊        return chat;
+┊   ┊ 60┊      }
+┊   ┊ 61┊    },
+┊   ┊ 62┊    addGroup: (obj, {userIds, groupName, groupPicture}) => {
+┊   ┊ 63┊      userIds.forEach(userId => {
+┊   ┊ 64┊        if (!users.find(user => user.id === Number(userId))) {
+┊   ┊ 65┊          throw new Error(`User ${userId} doesn't exist.`);
+┊   ┊ 66┊        }
+┊   ┊ 67┊      });
+┊   ┊ 68┊
+┊   ┊ 69┊      const id = (chats.length && chats[chats.length - 1].id + 1) || 1;
+┊   ┊ 70┊      const chat: ChatDb = {
+┊   ┊ 71┊        id,
+┊   ┊ 72┊        createdAt: moment().toDate(),
+┊   ┊ 73┊        name: groupName,
+┊   ┊ 74┊        picture: groupPicture || null,
+┊   ┊ 75┊        adminIds: [currentUser.id],
+┊   ┊ 76┊        ownerId: currentUser.id,
+┊   ┊ 77┊        allTimeMemberIds: [currentUser.id, ...userIds.map(id => Number(id))],
+┊   ┊ 78┊        listingMemberIds: [currentUser.id, ...userIds.map(id => Number(id))],
+┊   ┊ 79┊        actualGroupMemberIds: [currentUser.id, ...userIds.map(id => Number(id))],
+┊   ┊ 80┊        messages: [],
+┊   ┊ 81┊      };
+┊   ┊ 82┊      chats.push(chat);
+┊   ┊ 83┊      return chat;
+┊   ┊ 84┊    },
+┊   ┊ 85┊    updateGroup: (obj, {chatId, groupName, groupPicture}) => {
+┊   ┊ 86┊      const chat = chats.find(chat => chat.id === Number(chatId));
+┊   ┊ 87┊
+┊   ┊ 88┊      if (!chat) {
+┊   ┊ 89┊        throw new Error(`The chat ${chatId} doesn't exist.`);
+┊   ┊ 90┊      }
+┊   ┊ 91┊
+┊   ┊ 92┊      if (!chat.name) {
+┊   ┊ 93┊        throw new Error(`The chat ${chatId} is not a group.`);
+┊   ┊ 94┊      }
+┊   ┊ 95┊
+┊   ┊ 96┊      chat.name = groupName || chat.name;
+┊   ┊ 97┊      chat.picture = groupPicture || chat.picture;
+┊   ┊ 98┊
+┊   ┊ 99┊      return chat;
+┊   ┊100┊    },
+┊   ┊101┊    removeChat: (obj, {chatId}) => {
+┊   ┊102┊      const chat = chats.find(chat => chat.id === Number(chatId));
+┊   ┊103┊
+┊   ┊104┊      if (!chat) {
+┊   ┊105┊        throw new Error(`The chat ${chatId} doesn't exist.`);
+┊   ┊106┊      }
+┊   ┊107┊
+┊   ┊108┊      if (!chat.name) {
+┊   ┊109┊        // Chat
+┊   ┊110┊        if (!chat.listingMemberIds.includes(currentUser.id)) {
+┊   ┊111┊          throw new Error(`The user is not a member of the chat ${chatId}.`);
+┊   ┊112┊        }
+┊   ┊113┊
+┊   ┊114┊        // Instead of chaining map and filter we can loop once using reduce
+┊   ┊115┊        const messages = chat.messages.reduce<MessageDb[]>((filtered, message) => {
+┊   ┊116┊          // Remove the current user from the message holders
+┊   ┊117┊          message.holderIds = message.holderIds.filter(holderId => holderId !== currentUser.id);
+┊   ┊118┊
+┊   ┊119┊          if (message.holderIds.length !== 0) {
+┊   ┊120┊            filtered.push(message);
+┊   ┊121┊          } // else discard the message
+┊   ┊122┊
+┊   ┊123┊          return filtered;
+┊   ┊124┊        }, []);
+┊   ┊125┊
+┊   ┊126┊        // Remove the current user from who gets the chat listed. The chat will no longer appear in his list
+┊   ┊127┊        const listingMemberIds = chat.listingMemberIds.filter(listingId => listingId !== currentUser.id);
+┊   ┊128┊
+┊   ┊129┊        // Check how many members are left
+┊   ┊130┊        if (listingMemberIds.length === 0) {
+┊   ┊131┊          // Delete the chat
+┊   ┊132┊          chats = chats.filter(chat => chat.id !== Number(chatId));
+┊   ┊133┊        } else {
+┊   ┊134┊          // Update the chat
+┊   ┊135┊          chats = chats.map(chat => {
+┊   ┊136┊            if (chat.id === Number(chatId)) {
+┊   ┊137┊              chat = {...chat, listingMemberIds, messages};
+┊   ┊138┊            }
+┊   ┊139┊            return chat;
+┊   ┊140┊          });
+┊   ┊141┊        }
+┊   ┊142┊        return chatId;
+┊   ┊143┊      } else {
+┊   ┊144┊        // Group
+┊   ┊145┊        if (chat.ownerId !== currentUser.id) {
+┊   ┊146┊          throw new Error(`Group ${chatId} is not owned by the user.`);
+┊   ┊147┊        }
+┊   ┊148┊
+┊   ┊149┊        // Instead of chaining map and filter we can loop once using reduce
+┊   ┊150┊        const messages = chat.messages.reduce<MessageDb[]>((filtered, message) => {
+┊   ┊151┊          // Remove the current user from the message holders
+┊   ┊152┊          message.holderIds = message.holderIds.filter(holderId => holderId !== currentUser.id);
+┊   ┊153┊
+┊   ┊154┊          if (message.holderIds.length !== 0) {
+┊   ┊155┊            filtered.push(message);
+┊   ┊156┊          } // else discard the message
+┊   ┊157┊
+┊   ┊158┊          return filtered;
+┊   ┊159┊        }, []);
+┊   ┊160┊
+┊   ┊161┊        // Remove the current user from who gets the group listed. The group will no longer appear in his list
+┊   ┊162┊        const listingMemberIds = chat.listingMemberIds.filter(listingId => listingId !== currentUser.id);
+┊   ┊163┊
+┊   ┊164┊        // Check how many members (including previous ones who can still access old messages) are left
+┊   ┊165┊        if (listingMemberIds.length === 0) {
+┊   ┊166┊          // Remove the group
+┊   ┊167┊          chats = chats.filter(chat => chat.id !== Number(chatId));
+┊   ┊168┊        } else {
+┊   ┊169┊          // Update the group
+┊   ┊170┊
+┊   ┊171┊          // Remove the current user from the chat members. He is no longer a member of the group
+┊   ┊172┊          const actualGroupMemberIds = chat.actualGroupMemberIds!.filter(memberId => memberId !== currentUser.id);
+┊   ┊173┊          // Remove the current user from the chat admins
+┊   ┊174┊          const adminIds = chat.adminIds!.filter(memberId => memberId !== currentUser.id);
+┊   ┊175┊          // Set the owner id to be null. A null owner means the group is read-only
+┊   ┊176┊          let ownerId: number | null = null;
+┊   ┊177┊
+┊   ┊178┊          // Check if there is any admin left
+┊   ┊179┊          if (adminIds!.length) {
+┊   ┊180┊            // Pick an admin as the new owner. The group is no longer read-only
+┊   ┊181┊            ownerId = chat.adminIds![0];
+┊   ┊182┊          }
+┊   ┊183┊
+┊   ┊184┊          chats = chats.map(chat => {
+┊   ┊185┊            if (chat.id === Number(chatId)) {
+┊   ┊186┊              chat = {...chat, messages, listingMemberIds, actualGroupMemberIds, adminIds, ownerId};
+┊   ┊187┊            }
+┊   ┊188┊            return chat;
+┊   ┊189┊          });
+┊   ┊190┊        }
+┊   ┊191┊        return chatId;
+┊   ┊192┊      }
+┊   ┊193┊    },
+┊   ┊194┊    addMessage: (obj, {chatId, content}) => {
+┊   ┊195┊      if (content === null || content === '') {
+┊   ┊196┊        throw new Error(`Cannot add empty or null messages.`);
+┊   ┊197┊      }
+┊   ┊198┊
+┊   ┊199┊      let chat = chats.find(chat => chat.id === Number(chatId));
+┊   ┊200┊
+┊   ┊201┊      if (!chat) {
+┊   ┊202┊        throw new Error(`Cannot find chat ${chatId}.`);
+┊   ┊203┊      }
+┊   ┊204┊
+┊   ┊205┊      let holderIds = chat.listingMemberIds;
+┊   ┊206┊
+┊   ┊207┊      if (!chat.name) {
+┊   ┊208┊        // Chat
+┊   ┊209┊        if (!chat.listingMemberIds.find(listingId => listingId === currentUser.id)) {
+┊   ┊210┊          throw new Error(`The chat ${chatId} must be listed for the current user before adding a message.`);
+┊   ┊211┊        }
+┊   ┊212┊
+┊   ┊213┊        // Receiver's user
+┊   ┊214┊        const receiverId = chat.allTimeMemberIds.find(userId => userId !== currentUser.id);
+┊   ┊215┊
+┊   ┊216┊        if (!receiverId) {
+┊   ┊217┊          throw new Error(`Cannot find receiver's user.`);
+┊   ┊218┊        }
+┊   ┊219┊
+┊   ┊220┊        if (!chat.listingMemberIds.find(listingId => listingId === receiverId)) {
+┊   ┊221┊          // Chat is not listed for the receiver user. Add him to the listingMemberIds
+┊   ┊222┊          chat.listingMemberIds = chat.listingMemberIds.concat(receiverId);
+┊   ┊223┊
+┊   ┊224┊          holderIds = chat.listingMemberIds;
+┊   ┊225┊        }
+┊   ┊226┊      } else {
+┊   ┊227┊        // Group
+┊   ┊228┊        if (!chat.actualGroupMemberIds!.find(memberId => memberId === currentUser.id)) {
+┊   ┊229┊          throw new Error(`The user is not a member of the group ${chatId}. Cannot add message.`);
+┊   ┊230┊        }
+┊   ┊231┊
+┊   ┊232┊        holderIds = chat.actualGroupMemberIds!;
+┊   ┊233┊      }
+┊   ┊234┊
+┊   ┊235┊      const id = (chat.messages.length && chat.messages[chat.messages.length - 1].id + 1) || 1;
+┊   ┊236┊
+┊   ┊237┊      let recipients: RecipientDb[] = [];
+┊   ┊238┊
+┊   ┊239┊      holderIds.forEach(holderId => {
+┊   ┊240┊        if (holderId !== currentUser.id) {
+┊   ┊241┊          recipients.push({
+┊   ┊242┊            userId: holderId,
+┊   ┊243┊            messageId: id,
+┊   ┊244┊            chatId: Number(chatId),
+┊   ┊245┊            receivedAt: null,
+┊   ┊246┊            readAt: null,
+┊   ┊247┊          });
+┊   ┊248┊        }
+┊   ┊249┊      });
+┊   ┊250┊
+┊   ┊251┊      const message: MessageDb = {
+┊   ┊252┊        id,
+┊   ┊253┊        chatId: Number(chatId),
+┊   ┊254┊        senderId: currentUser.id,
+┊   ┊255┊        content,
+┊   ┊256┊        createdAt: moment().toDate(),
+┊   ┊257┊        type: MessageType.TEXT,
+┊   ┊258┊        recipients,
+┊   ┊259┊        holderIds,
+┊   ┊260┊      };
+┊   ┊261┊
+┊   ┊262┊      chats = chats.map(chat => {
+┊   ┊263┊        if (chat.id === Number(chatId)) {
+┊   ┊264┊          chat = {...chat, messages: chat.messages.concat(message)}
+┊   ┊265┊        }
+┊   ┊266┊        return chat;
+┊   ┊267┊      });
+┊   ┊268┊
+┊   ┊269┊      return message;
+┊   ┊270┊    },
+┊   ┊271┊    removeMessages: (obj, {chatId, messageIds, all}) => {
+┊   ┊272┊      const chat = chats.find(chat => chat.id === Number(chatId));
+┊   ┊273┊
+┊   ┊274┊      if (!chat) {
+┊   ┊275┊        throw new Error(`Cannot find chat ${chatId}.`);
+┊   ┊276┊      }
+┊   ┊277┊
+┊   ┊278┊      if (!chat.listingMemberIds.find(listingId => listingId === currentUser.id)) {
+┊   ┊279┊        throw new Error(`The chat/group ${chatId} is not listed for the current user, so there is nothing to delete.`);
+┊   ┊280┊      }
+┊   ┊281┊
+┊   ┊282┊      if (all && messageIds) {
+┊   ┊283┊        throw new Error(`Cannot specify both 'all' and 'messageIds'.`);
+┊   ┊284┊      }
+┊   ┊285┊
+┊   ┊286┊      let deletedIds: string[] = [];
+┊   ┊287┊      chats = chats.map(chat => {
+┊   ┊288┊        if (chat.id === Number(chatId)) {
+┊   ┊289┊          // Instead of chaining map and filter we can loop once using reduce
+┊   ┊290┊          const messages = chat.messages.reduce<MessageDb[]>((filtered, message) => {
+┊   ┊291┊            if (all || messageIds!.map(Number).includes(message.id)) {
+┊   ┊292┊              deletedIds.push(String(message.id));
+┊   ┊293┊              // Remove the current user from the message holders
+┊   ┊294┊              message.holderIds = message.holderIds.filter(holderId => holderId !== currentUser.id);
+┊   ┊295┊            }
+┊   ┊296┊
+┊   ┊297┊            if (message.holderIds.length !== 0) {
+┊   ┊298┊              filtered.push(message);
+┊   ┊299┊            } // else discard the message
+┊   ┊300┊
+┊   ┊301┊            return filtered;
+┊   ┊302┊          }, []);
+┊   ┊303┊          chat = {...chat, messages};
+┊   ┊304┊        }
+┊   ┊305┊        return chat;
+┊   ┊306┊      });
+┊   ┊307┊      return deletedIds;
+┊   ┊308┊    },
+┊   ┊309┊  },
 ┊ 17┊310┊  Chat: {
 ┊ 18┊311┊    name: (chat) => chat.name ? chat.name : users
 ┊ 19┊312┊      .find(user => user.id === chat.allTimeMemberIds.find(userId => userId !== currentUser.id))!.name,
```

##### Changed schema&#x2F;typeDefs.ts
```diff
@@ -73,4 +73,22 @@
 ┊73┊73┊    picture: String
 ┊74┊74┊    phone: String
 ┊75┊75┊  }
+┊  ┊76┊
+┊  ┊77┊  type Mutation {
+┊  ┊78┊    updateUser(name: String, picture: String): User!
+┊  ┊79┊    addChat(userId: ID!): Chat
+┊  ┊80┊    addGroup(userIds: [ID!]!, groupName: String!, groupPicture: String): Chat
+┊  ┊81┊    updateGroup(chatId: ID!, groupName: String, groupPicture: String): Chat
+┊  ┊82┊    removeChat(chatId: ID!): ID
+┊  ┊83┊    addMessage(chatId: ID!, content: String!): Message
+┊  ┊84┊    removeMessages(chatId: ID!, messageIds: [ID], all: Boolean): [ID]
+┊  ┊85┊    addMembers(groupId: ID!, userIds: [ID!]!): [ID]
+┊  ┊86┊    removeMembers(groupId: ID!, userIds: [ID!]!): [ID]
+┊  ┊87┊    addAdmins(groupId: ID!, userIds: [ID!]!): [ID]
+┊  ┊88┊    removeAdmins(groupId: ID!, userIds: [ID!]!): [ID]
+┊  ┊89┊    setGroupName(groupId: ID!): String
+┊  ┊90┊    setGroupPicture(groupId: ID!): String
+┊  ┊91┊    markAsReceived(chatId: ID!): Boolean
+┊  ┊92┊    markAsRead(chatId: ID!): Boolean
+┊  ┊93┊  }
 ┊76┊94┊`;
```

##### Changed types.d.ts
```diff
@@ -98,6 +98,38 @@
 ┊ 98┊ 98┊  readAt?: Maybe<Date>;
 ┊ 99┊ 99┊}
 ┊100┊100┊
+┊   ┊101┊export interface Mutation {
+┊   ┊102┊  updateUser: User;
+┊   ┊103┊
+┊   ┊104┊  addChat?: Maybe<Chat>;
+┊   ┊105┊
+┊   ┊106┊  addGroup?: Maybe<Chat>;
+┊   ┊107┊
+┊   ┊108┊  updateGroup?: Maybe<Chat>;
+┊   ┊109┊
+┊   ┊110┊  removeChat?: Maybe<string>;
+┊   ┊111┊
+┊   ┊112┊  addMessage?: Maybe<Message>;
+┊   ┊113┊
+┊   ┊114┊  removeMessages?: Maybe<(Maybe<string>)[]>;
+┊   ┊115┊
+┊   ┊116┊  addMembers?: Maybe<(Maybe<string>)[]>;
+┊   ┊117┊
+┊   ┊118┊  removeMembers?: Maybe<(Maybe<string>)[]>;
+┊   ┊119┊
+┊   ┊120┊  addAdmins?: Maybe<(Maybe<string>)[]>;
+┊   ┊121┊
+┊   ┊122┊  removeAdmins?: Maybe<(Maybe<string>)[]>;
+┊   ┊123┊
+┊   ┊124┊  setGroupName?: Maybe<string>;
+┊   ┊125┊
+┊   ┊126┊  setGroupPicture?: Maybe<string>;
+┊   ┊127┊
+┊   ┊128┊  markAsReceived?: Maybe<boolean>;
+┊   ┊129┊
+┊   ┊130┊  markAsRead?: Maybe<boolean>;
+┊   ┊131┊}
+┊   ┊132┊
 ┊101┊133┊// ====================================================
 ┊102┊134┊// Arguments
 ┊103┊135┊// ====================================================
```
```diff
@@ -108,6 +140,75 @@
 ┊108┊140┊export interface MessagesChatArgs {
 ┊109┊141┊  amount?: Maybe<number>;
 ┊110┊142┊}
+┊   ┊143┊export interface UpdateUserMutationArgs {
+┊   ┊144┊  name?: Maybe<string>;
+┊   ┊145┊
+┊   ┊146┊  picture?: Maybe<string>;
+┊   ┊147┊}
+┊   ┊148┊export interface AddChatMutationArgs {
+┊   ┊149┊  userId: string;
+┊   ┊150┊}
+┊   ┊151┊export interface AddGroupMutationArgs {
+┊   ┊152┊  userIds: string[];
+┊   ┊153┊
+┊   ┊154┊  groupName: string;
+┊   ┊155┊
+┊   ┊156┊  groupPicture?: Maybe<string>;
+┊   ┊157┊}
+┊   ┊158┊export interface UpdateGroupMutationArgs {
+┊   ┊159┊  chatId: string;
+┊   ┊160┊
+┊   ┊161┊  groupName?: Maybe<string>;
+┊   ┊162┊
+┊   ┊163┊  groupPicture?: Maybe<string>;
+┊   ┊164┊}
+┊   ┊165┊export interface RemoveChatMutationArgs {
+┊   ┊166┊  chatId: string;
+┊   ┊167┊}
+┊   ┊168┊export interface AddMessageMutationArgs {
+┊   ┊169┊  chatId: string;
+┊   ┊170┊
+┊   ┊171┊  content: string;
+┊   ┊172┊}
+┊   ┊173┊export interface RemoveMessagesMutationArgs {
+┊   ┊174┊  chatId: string;
+┊   ┊175┊
+┊   ┊176┊  messageIds?: Maybe<(Maybe<string>)[]>;
+┊   ┊177┊
+┊   ┊178┊  all?: Maybe<boolean>;
+┊   ┊179┊}
+┊   ┊180┊export interface AddMembersMutationArgs {
+┊   ┊181┊  groupId: string;
+┊   ┊182┊
+┊   ┊183┊  userIds: string[];
+┊   ┊184┊}
+┊   ┊185┊export interface RemoveMembersMutationArgs {
+┊   ┊186┊  groupId: string;
+┊   ┊187┊
+┊   ┊188┊  userIds: string[];
+┊   ┊189┊}
+┊   ┊190┊export interface AddAdminsMutationArgs {
+┊   ┊191┊  groupId: string;
+┊   ┊192┊
+┊   ┊193┊  userIds: string[];
+┊   ┊194┊}
+┊   ┊195┊export interface RemoveAdminsMutationArgs {
+┊   ┊196┊  groupId: string;
+┊   ┊197┊
+┊   ┊198┊  userIds: string[];
+┊   ┊199┊}
+┊   ┊200┊export interface SetGroupNameMutationArgs {
+┊   ┊201┊  groupId: string;
+┊   ┊202┊}
+┊   ┊203┊export interface SetGroupPictureMutationArgs {
+┊   ┊204┊  groupId: string;
+┊   ┊205┊}
+┊   ┊206┊export interface MarkAsReceivedMutationArgs {
+┊   ┊207┊  chatId: string;
+┊   ┊208┊}
+┊   ┊209┊export interface MarkAsReadMutationArgs {
+┊   ┊210┊  chatId: string;
+┊   ┊211┊}
 ┊111┊212┊
 ┊112┊213┊import {
 ┊113┊214┊  GraphQLResolveInfo,
```
```diff
@@ -454,6 +555,227 @@
 ┊454┊555┊  > = Resolver<R, Parent, TContext>;
 ┊455┊556┊}
 ┊456┊557┊
+┊   ┊558┊export namespace MutationResolvers {
+┊   ┊559┊  export interface Resolvers<TContext = {}, TypeParent = {}> {
+┊   ┊560┊    updateUser?: UpdateUserResolver<UserDb, TypeParent, TContext>;
+┊   ┊561┊
+┊   ┊562┊    addChat?: AddChatResolver<Maybe<ChatDb>, TypeParent, TContext>;
+┊   ┊563┊
+┊   ┊564┊    addGroup?: AddGroupResolver<Maybe<ChatDb>, TypeParent, TContext>;
+┊   ┊565┊
+┊   ┊566┊    updateGroup?: UpdateGroupResolver<Maybe<ChatDb>, TypeParent, TContext>;
+┊   ┊567┊
+┊   ┊568┊    removeChat?: RemoveChatResolver<Maybe<string>, TypeParent, TContext>;
+┊   ┊569┊
+┊   ┊570┊    addMessage?: AddMessageResolver<Maybe<MessageDb>, TypeParent, TContext>;
+┊   ┊571┊
+┊   ┊572┊    removeMessages?: RemoveMessagesResolver<
+┊   ┊573┊      Maybe<(Maybe<string>)[]>,
+┊   ┊574┊      TypeParent,
+┊   ┊575┊      TContext
+┊   ┊576┊    >;
+┊   ┊577┊
+┊   ┊578┊    addMembers?: AddMembersResolver<
+┊   ┊579┊      Maybe<(Maybe<string>)[]>,
+┊   ┊580┊      TypeParent,
+┊   ┊581┊      TContext
+┊   ┊582┊    >;
+┊   ┊583┊
+┊   ┊584┊    removeMembers?: RemoveMembersResolver<
+┊   ┊585┊      Maybe<(Maybe<string>)[]>,
+┊   ┊586┊      TypeParent,
+┊   ┊587┊      TContext
+┊   ┊588┊    >;
+┊   ┊589┊
+┊   ┊590┊    addAdmins?: AddAdminsResolver<
+┊   ┊591┊      Maybe<(Maybe<string>)[]>,
+┊   ┊592┊      TypeParent,
+┊   ┊593┊      TContext
+┊   ┊594┊    >;
+┊   ┊595┊
+┊   ┊596┊    removeAdmins?: RemoveAdminsResolver<
+┊   ┊597┊      Maybe<(Maybe<string>)[]>,
+┊   ┊598┊      TypeParent,
+┊   ┊599┊      TContext
+┊   ┊600┊    >;
+┊   ┊601┊
+┊   ┊602┊    setGroupName?: SetGroupNameResolver<Maybe<string>, TypeParent, TContext>;
+┊   ┊603┊
+┊   ┊604┊    setGroupPicture?: SetGroupPictureResolver<
+┊   ┊605┊      Maybe<string>,
+┊   ┊606┊      TypeParent,
+┊   ┊607┊      TContext
+┊   ┊608┊    >;
+┊   ┊609┊
+┊   ┊610┊    markAsReceived?: MarkAsReceivedResolver<
+┊   ┊611┊      Maybe<boolean>,
+┊   ┊612┊      TypeParent,
+┊   ┊613┊      TContext
+┊   ┊614┊    >;
+┊   ┊615┊
+┊   ┊616┊    markAsRead?: MarkAsReadResolver<Maybe<boolean>, TypeParent, TContext>;
+┊   ┊617┊  }
+┊   ┊618┊
+┊   ┊619┊  export type UpdateUserResolver<
+┊   ┊620┊    R = UserDb,
+┊   ┊621┊    Parent = {},
+┊   ┊622┊    TContext = {}
+┊   ┊623┊  > = Resolver<R, Parent, TContext, UpdateUserArgs>;
+┊   ┊624┊  export interface UpdateUserArgs {
+┊   ┊625┊    name?: Maybe<string>;
+┊   ┊626┊
+┊   ┊627┊    picture?: Maybe<string>;
+┊   ┊628┊  }
+┊   ┊629┊
+┊   ┊630┊  export type AddChatResolver<
+┊   ┊631┊    R = Maybe<ChatDb>,
+┊   ┊632┊    Parent = {},
+┊   ┊633┊    TContext = {}
+┊   ┊634┊  > = Resolver<R, Parent, TContext, AddChatArgs>;
+┊   ┊635┊  export interface AddChatArgs {
+┊   ┊636┊    userId: string;
+┊   ┊637┊  }
+┊   ┊638┊
+┊   ┊639┊  export type AddGroupResolver<
+┊   ┊640┊    R = Maybe<ChatDb>,
+┊   ┊641┊    Parent = {},
+┊   ┊642┊    TContext = {}
+┊   ┊643┊  > = Resolver<R, Parent, TContext, AddGroupArgs>;
+┊   ┊644┊  export interface AddGroupArgs {
+┊   ┊645┊    userIds: string[];
+┊   ┊646┊
+┊   ┊647┊    groupName: string;
+┊   ┊648┊
+┊   ┊649┊    groupPicture?: Maybe<string>;
+┊   ┊650┊  }
+┊   ┊651┊
+┊   ┊652┊  export type UpdateGroupResolver<
+┊   ┊653┊    R = Maybe<ChatDb>,
+┊   ┊654┊    Parent = {},
+┊   ┊655┊    TContext = {}
+┊   ┊656┊  > = Resolver<R, Parent, TContext, UpdateGroupArgs>;
+┊   ┊657┊  export interface UpdateGroupArgs {
+┊   ┊658┊    chatId: string;
+┊   ┊659┊
+┊   ┊660┊    groupName?: Maybe<string>;
+┊   ┊661┊
+┊   ┊662┊    groupPicture?: Maybe<string>;
+┊   ┊663┊  }
+┊   ┊664┊
+┊   ┊665┊  export type RemoveChatResolver<
+┊   ┊666┊    R = Maybe<string>,
+┊   ┊667┊    Parent = {},
+┊   ┊668┊    TContext = {}
+┊   ┊669┊  > = Resolver<R, Parent, TContext, RemoveChatArgs>;
+┊   ┊670┊  export interface RemoveChatArgs {
+┊   ┊671┊    chatId: string;
+┊   ┊672┊  }
+┊   ┊673┊
+┊   ┊674┊  export type AddMessageResolver<
+┊   ┊675┊    R = Maybe<MessageDb>,
+┊   ┊676┊    Parent = {},
+┊   ┊677┊    TContext = {}
+┊   ┊678┊  > = Resolver<R, Parent, TContext, AddMessageArgs>;
+┊   ┊679┊  export interface AddMessageArgs {
+┊   ┊680┊    chatId: string;
+┊   ┊681┊
+┊   ┊682┊    content: string;
+┊   ┊683┊  }
+┊   ┊684┊
+┊   ┊685┊  export type RemoveMessagesResolver<
+┊   ┊686┊    R = Maybe<(Maybe<string>)[]>,
+┊   ┊687┊    Parent = {},
+┊   ┊688┊    TContext = {}
+┊   ┊689┊  > = Resolver<R, Parent, TContext, RemoveMessagesArgs>;
+┊   ┊690┊  export interface RemoveMessagesArgs {
+┊   ┊691┊    chatId: string;
+┊   ┊692┊
+┊   ┊693┊    messageIds?: Maybe<(Maybe<string>)[]>;
+┊   ┊694┊
+┊   ┊695┊    all?: Maybe<boolean>;
+┊   ┊696┊  }
+┊   ┊697┊
+┊   ┊698┊  export type AddMembersResolver<
+┊   ┊699┊    R = Maybe<(Maybe<string>)[]>,
+┊   ┊700┊    Parent = {},
+┊   ┊701┊    TContext = {}
+┊   ┊702┊  > = Resolver<R, Parent, TContext, AddMembersArgs>;
+┊   ┊703┊  export interface AddMembersArgs {
+┊   ┊704┊    groupId: string;
+┊   ┊705┊
+┊   ┊706┊    userIds: string[];
+┊   ┊707┊  }
+┊   ┊708┊
+┊   ┊709┊  export type RemoveMembersResolver<
+┊   ┊710┊    R = Maybe<(Maybe<string>)[]>,
+┊   ┊711┊    Parent = {},
+┊   ┊712┊    TContext = {}
+┊   ┊713┊  > = Resolver<R, Parent, TContext, RemoveMembersArgs>;
+┊   ┊714┊  export interface RemoveMembersArgs {
+┊   ┊715┊    groupId: string;
+┊   ┊716┊
+┊   ┊717┊    userIds: string[];
+┊   ┊718┊  }
+┊   ┊719┊
+┊   ┊720┊  export type AddAdminsResolver<
+┊   ┊721┊    R = Maybe<(Maybe<string>)[]>,
+┊   ┊722┊    Parent = {},
+┊   ┊723┊    TContext = {}
+┊   ┊724┊  > = Resolver<R, Parent, TContext, AddAdminsArgs>;
+┊   ┊725┊  export interface AddAdminsArgs {
+┊   ┊726┊    groupId: string;
+┊   ┊727┊
+┊   ┊728┊    userIds: string[];
+┊   ┊729┊  }
+┊   ┊730┊
+┊   ┊731┊  export type RemoveAdminsResolver<
+┊   ┊732┊    R = Maybe<(Maybe<string>)[]>,
+┊   ┊733┊    Parent = {},
+┊   ┊734┊    TContext = {}
+┊   ┊735┊  > = Resolver<R, Parent, TContext, RemoveAdminsArgs>;
+┊   ┊736┊  export interface RemoveAdminsArgs {
+┊   ┊737┊    groupId: string;
+┊   ┊738┊
+┊   ┊739┊    userIds: string[];
+┊   ┊740┊  }
+┊   ┊741┊
+┊   ┊742┊  export type SetGroupNameResolver<
+┊   ┊743┊    R = Maybe<string>,
+┊   ┊744┊    Parent = {},
+┊   ┊745┊    TContext = {}
+┊   ┊746┊  > = Resolver<R, Parent, TContext, SetGroupNameArgs>;
+┊   ┊747┊  export interface SetGroupNameArgs {
+┊   ┊748┊    groupId: string;
+┊   ┊749┊  }
+┊   ┊750┊
+┊   ┊751┊  export type SetGroupPictureResolver<
+┊   ┊752┊    R = Maybe<string>,
+┊   ┊753┊    Parent = {},
+┊   ┊754┊    TContext = {}
+┊   ┊755┊  > = Resolver<R, Parent, TContext, SetGroupPictureArgs>;
+┊   ┊756┊  export interface SetGroupPictureArgs {
+┊   ┊757┊    groupId: string;
+┊   ┊758┊  }
+┊   ┊759┊
+┊   ┊760┊  export type MarkAsReceivedResolver<
+┊   ┊761┊    R = Maybe<boolean>,
+┊   ┊762┊    Parent = {},
+┊   ┊763┊    TContext = {}
+┊   ┊764┊  > = Resolver<R, Parent, TContext, MarkAsReceivedArgs>;
+┊   ┊765┊  export interface MarkAsReceivedArgs {
+┊   ┊766┊    chatId: string;
+┊   ┊767┊  }
+┊   ┊768┊
+┊   ┊769┊  export type MarkAsReadResolver<
+┊   ┊770┊    R = Maybe<boolean>,
+┊   ┊771┊    Parent = {},
+┊   ┊772┊    TContext = {}
+┊   ┊773┊  > = Resolver<R, Parent, TContext, MarkAsReadArgs>;
+┊   ┊774┊  export interface MarkAsReadArgs {
+┊   ┊775┊    chatId: string;
+┊   ┊776┊  }
+┊   ┊777┊}
+┊   ┊778┊
 ┊457┊779┊/** Directs the executor to skip this field or fragment when the `if` argument is true. */
 ┊458┊780┊export type SkipDirectiveResolver<Result> = DirectiveResolverFn<
 ┊459┊781┊  Result,
```
```diff
@@ -497,6 +819,7 @@
 ┊497┊819┊  Chat?: ChatResolvers.Resolvers<TContext>;
 ┊498┊820┊  Message?: MessageResolvers.Resolvers<TContext>;
 ┊499┊821┊  Recipient?: RecipientResolvers.Resolvers<TContext>;
+┊   ┊822┊  Mutation?: MutationResolvers.Resolvers<TContext>;
 ┊500┊823┊  Date?: GraphQLScalarType;
 ┊501┊824┊}
```

[}]: #

    $ npm run generator

[{]: <helper> (diffStep "3.3")

#### Step 3.3: NOT FOUND!

[}]: #



[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](https://github.com/Urigo/WhatsApp-Clone-Server/tree/master@2.0.2/.tortilla/manuals/views/step2.md) | [Next Step >](https://github.com/Urigo/WhatsApp-Clone-Server/tree/master@2.0.2/.tortilla/manuals/views/step4.md) |
|:--------------------------------|--------------------------------:|

[}]: #
