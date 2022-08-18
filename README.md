# Angular Refactoring

## Learning Goals

- Refactor the Angular app from the previous week.

## Introduction

Let's address something that you may have noticed from the previous section. We
created a few models to make our views read data from our application code
instead of having it hardcoded in the views themselves. For simplicity’s sake,
we created those models without actually creating types. This might have been
expedient, but it didn't take advantage of TypeScript's type functionality and
shouldn't be the way we write code moving forward.

Let's add proper types to our application code, starting with the "User" type we
used in the `contact-list` and `contact` components.

First, we create a `user.model.ts` file in our root directory (because we want
other components to be able to use the same type) and define the (very simple)
`User` type there:

```typescript
export class User {
  constructor(public firstName: string) {}
}
```

Then we change the declaration of the `users` array in our contact list
component in `contact-list-component.component.ts` to give it a type:

```typescript
import { Component, OnInit } from "@angular/core";
import { User } from "../../user.model";

@Component({
  selector: "app-contact-list-component",
  templateUrl: "./contact-list-component.component.html",
  styleUrls: ["./contact-list-component.component.css"],
})
export class ContactListComponentComponent implements OnInit {
  users: User[] = [
    { firstName: "Aurelie" },
    { firstName: "James" },
    { firstName: "Jessica" },
    { firstName: "Ludovic" },
    { firstName: "Maria" },
  ];

  constructor() {}

  ngOnInit(): void {}
}
```

Then we can add code to our constructor to demonstrate how the use of types
helps the IDE give us auto-complete functionality:

![Typescript auto-complete](https://curriculum-content.s3.amazonaws.com/java-mod-8/ts-type-autocomplete.png)

This will also work to tell us where we're not trying to use a type improperly.

We also need to update the declaration of the `user` variable in the contact
component in `contact-component.component.ts`:

```typescript
import { Component, Input, OnInit } from "@angular/core";
import { User } from "src/app/user.model";

@Component({
  selector: "app-contact-component",
  templateUrl: "./contact-component.component.html",
  styleUrls: ["./contact-component.component.css"],
})
export class ContactComponentComponent implements OnInit {
  @Input() user: User;

  constructor() {}

  ngOnInit(): void {}
}
```

When you run the application with the updated code, you might notice an error in
the console because the previous version of the contact list component had 2
references to the contact component, on of which did not correctly pass in a
value for the user:

```html
<div class="container">
  <div class="row">
    <div class="col-12">
      <h3>Contact List</h3>
    </div>
  </div>
  <div class="row" *ngFor="let user of users">
    <div class="col-12 border p-3">
      <app-contact-component [user]="user"></app-contact-component>
    </div>
  </div>

  <!-- Remove this component -->
  <div class="row">
    <div class="col-12 border p-3">
      <app-contact-component></app-contact-component>
    </div>
  </div>
  <!-- end of component that needs to be removed -->

  <div class="row">
    <div class="col-9"></div>
    <div class="col-3 p-3">
      <div class="float-end">
        <button class="btn btn-primary">Start Message</button>
      </div>
    </div>
  </div>
</div>
```

We also need to update the header component in `header-component.component.ts`
to use the `User` type:

```typescript
import { Component, OnInit } from "@angular/core";
import { User } from "../user.model";

@Component({
  selector: "app-header-component",
  templateUrl: "./header-component.component.html",
  styleUrls: ["./header-component.component.css"],
})
export class HeaderComponentComponent implements OnInit {
  activeUser: User = {
    firstName: "Maria",
  };

  constructor() {}

  ngOnInit(): void {}
}
```

Let's tackle a slightly more complex type, from our conversation control
component in `conversation-control-component.component.ts`. Our `conversations`
array was defined like this:

```typescript
conversations = [
  {
    id: 1,
    users: [
      { firstName: "Claire" },
      { firstName: "Ludovic" },
      { firstName: "Jessica" },
    ],
  },
  {
    id: 2,
    users: [{ firstName: "Claire" }, { firstName: "James" }],
  },
  {
    id: 3,
    users: [
      { firstName: "Claire" },
      { firstName: "Aurelie" },
      { firstName: "James" },
      { firstName: "Jessica" },
    ],
  },
];
```

We discussed that a "conversation" is something with the following properties:

1. An id
2. The list of users in the conversation

Since we now have a `User` type, the `Conversation` type can easily be defined
in `conversation.model.ts` as follows:

```typescript
import { User } from "./user.model";

export class Conversation {
  constructor(public id: number, public users: User[]) {}
}
```

You can already see how the use of types also helps with the complexity of an
application as we build it up. If we wanted to add a new property to the `User`
type, we would simply modify it in `user.model.ts` file we created earlier, and
our `Conversation` type would automatically have access to the new properties.

> Note: you might have to make new properties optional if you're adding them to
> an existing type but do not want to or cannot update all the places where that
> type is used at the time.

Now we can use our new `Conversation` type in
`conversation-control-component.component.ts`:

```typescript
import { Component, OnInit } from "@angular/core";
import { Conversation } from "../../conversation.model";

@Component({
  selector: "app-conversation-control-component",
  templateUrl: "./conversation-control-component.component.html",
  styleUrls: ["./conversation-control-component.component.css"],
})
export class ConversationControlComponentComponent implements OnInit {
  conversations: Conversation[] = [
    {
      id: 1,
      users: [
        { firstName: "Claire" },
        { firstName: "Ludovic" },
        { firstName: "Jessica" },
      ],
    },
    {
      id: 2,
      users: [{ firstName: "Claire" }, { firstName: "James" }],
    },
    {
      id: 3,
      users: [
        { firstName: "Claire" },
        { firstName: "Aurelie" },
        { firstName: "James" },
        { firstName: "Jessica" },
      ],
    },
  ];

  constructor() {}

  ngOnInit(): void {}
}
```

Another advantage of using types is that we can identify where we're using
similar data structures, associate them with the same type and ensure
consistency in their definition and usage. In our messaging application, this
applies to use of the "Message" type for both the sender messages and the user
messages.

Let's start by creating a `Message` type in `message.model.ts`:

```typescript
import { User } from "./user.model";

export class Message {
  constructor(
    public sender: User,
    public text: string,
    public conversationId: number,
    public sequenceNumber: number
  ) {}
}
```

Then we change both the sender message and the user message:

- `sender-message-component.component.ts`:

```typescript
import { Component, Input, OnInit } from "@angular/core";
import { Message } from "src/app/message.model";

@Component({
  selector: "app-sender-message-component",
  templateUrl: "./sender-message-component.component.html",
  styleUrls: ["./sender-message-component.component.css"],
})
export class SenderMessageComponentComponent implements OnInit {
  @Input() message: Message = {
    sender: { firstName: "Ludovic" },
    text: "Message from Ludovic",
    conversationId: 1,
    sequenceNumber: 0,
  };

  constructor() {}

  ngOnInit(): void {}
}
```

- `user-message-component.component.ts`:

```typescript
import { Component, Input, OnInit } from "@angular/core";
import { Message } from "src/app/message.model";

@Component({
  selector: "app-user-message-component",
  templateUrl: "./user-message-component.component.html",
  styleUrls: ["./user-message-component.component.css"],
})
export class UserMessageComponentComponent implements OnInit {
  @Input() message: Message = {
    sender: { firstName: "Ludovic" },
    text: "Message from Ludovic",
    conversationId: 1,
    sequenceNumber: 0,
  };

  constructor() {}

  ngOnInit(): void {}
}
```

And we also need to update the conversation thread component in
`conversation-thread-component.component.ts`:

```typescript
import { Component, OnInit } from "@angular/core";
import { Message } from "src/app/message.model";

@Component({
  selector: "app-conversation-thread-component",
  templateUrl: "./conversation-thread-component.component.html",
  styleUrls: ["./conversation-thread-component.component.css"],
})
export class ConversationThreadComponentComponent implements OnInit {
  senderMessages: Message[] = [
    {
      sender: { firstName: "Ludovic" },
      text: "Message from Ludovic",
      conversationId: 1,
      sequenceNumber: 0,
    },
    {
      sender: { firstName: "Jessica" },
      text: "Message from Jessica",
      conversationId: 1,
      sequenceNumber: 1,
    },
  ];

  userMessages: Message[] = [
    {
      sender: { firstName: "Aurelie" },
      text: "Message from Aurelie",
      conversationId: 1,
      sequenceNumber: 2,
    },
  ];
  constructor() {}

  ngOnInit(): void {}
}
```
