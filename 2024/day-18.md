**Author:** Tuna

---

# Jsonize with Custom Field Name in TypeScript

A step by step discovery of how to customize JSON serialization and deserialization in TypeScript.

![icon](https://iamtuna.org/assets/apple-touch-icon.png)

Original article:  
https://iamtuna.org/2024-12-14/jsonize-with-custom-field-name

---

### Introduction

As some of you may know, I created an ASCII drawing app called [MonoSketch](https://github.com/tuanchauict/MonoSketch). Initially developed in Kotlin/JS, the app’s architecture served us well for the early iterations. However, the UI became a bottleneck whenever I wanted to add more features. This led to the decision to rewrite MonoSketch in TypeScript.

Rewriting in TypeScript, with [Svelte](https://svelte.dev/) for the UI, offered several benefits: it made UI development easier, better integration with FrontEnd web technologies, and made adding new features simpler. However, one key challenge I faced was handling JSON serialization and deserialization in TypeScript, which is used for data persistence.

MonoSketch saves drawings as a custom format `.mono` file, which is essentially a JSON file with a custom structure:

- Field names are obfuscated for shorter file sizes (e.g., `text` is serialized as `t`).
- Some value types are serialized in a custom format (e.g., a `Point(left, right)` object is serialized as a single string in the format `"{left}|{right}"`).

These customizations were straightforward to implement in Kotlin, thanks to [Kotlin’s serialization library](https://kotlinlang.org/docs/serialization.html). However, TypeScript lacks a built-in mechanism for customizing JSON field names and serialization rules. Although there is a library called [class-transformer](https://github.com/typestack/class-transformer) that offers similar features to Kotlin’s serialization, I found it too complex for MonoSketch and was lazy to learn it. This led me to develop a custom solution for JSON serialization and deserialization in TypeScript, which I’ll share in this post.

#### Key Requirements

First, let’s dive into the requirements that shaped the design of the solution.

When designing a solution for MonoSketch, I identified three primary requirements that the system needed to fulfill:

1. Custom Field Names  
2. Custom Serialization and Deserialization of Values  
3. Support for Nested Objects  

Let’s address each requirement in detail.

---

### Custom Field Names

This requirement is simple: we need to map the field names in the TypeScript class to the field names in the JSON file. For example, the extra value of a text shape looks like this:

```ts
class TextShapeExtra {
  text: string;
  fontSize: number;
}
```

will be serialized as:

```json
{
  "t": "Hello",
  "fs": 14
}
```

A simple solution is we will add 2 methods like these:

```ts
class TextShapeExtra {
  text: string;
  fontSize: number;

  toJson(): any {
    return {
      t: this.text,
      fs: this.fontSize,
    };
  }

  static fromJson(json: any): TextShapeExtra {
    const extra = new TextShapeExtra();
    extra.text = json.t;
    extra.fontSize = json.fs;
    return extra;
  }
}
```

This solution is simple and straightforward, but it has a few drawbacks:

- It requires manual mapping of each field, which can be tedious for classes with many fields.
- It is error-prone, as a typo in the field name can lead to runtime errors.
- It is not scalable, as adding or removing fields requires updating the `toJson()` and `fromJson()` methods.
- It is not reusable, as the mapping logic is tightly coupled with the class implementation.

#### Solution: Decorators

In Kotlin, we can use the [@SerialName](https://kotlinlang.org/api/kotlinx.serialization/kotlinx-serialization-core/kotlinx.serialization/-serial-name/) annotation to customize field names. So, the solution I came up with for TypeScript was to create a decorator that mimics this behavior. The decorator would allow us to specify custom field names in the class definition, which would be used during serialization and deserialization.

Sample usage:

```ts
class TextShapeExtra {
  @SerialName("t")
  text: string;

  @SerialName("fs")
  fontSize: number;
}
```

`@SerialName` decorator is defined as follows:

```ts
export type JsonizableClass<T> = {
  new (...args: any[]): T;

  serialNames?: Map<string, string>;
  serializers?: Map<string, (value: any) => any>;
  deserializers?: Map<string, (value: any) => any>;

  fromJson?(this: JsonizableClass<T>, json: any): T;
};

export function SerialName(name: string): PropertyDecorator {
  return function <T>(
    target: Object,
    propertyKey: string | symbol,
  ): void {
    const constructor = target.constructor as JsonizableClass<T>;
    if (!constructor.serialNames) {
      constructor.serialNames = new Map();
    }
    constructor.serialNames.set(propertyKey.toString(), name);
  };
}
```

The `SerialName` decorator stores the custom field names in a static property called `serialNames` within the class. For the `TextShapeExtra` class, the `serialNames` property would look like this:

```ts
TextShapeExtra.serialNames = new Map([
  ["text", "t"],
  ["fontSize", "fs"],
]);
```

This property will be used to map the field names during serialization and deserialization.

With this decorator, we can now serialize and deserialize objects with custom field names without having to write custom mapping logic for each class. Of course, `toJson` and `fromJson` methods haven’t yet been implemented, but we will cover them in the later section.

---

### Custom Serialization and Deserialization of Values

The second requirement is to support custom serialization and deserialization of values. In the `Point` sample above, we need to serialize a `Point(left, right)` object as a string in the format `"{left}|{right}"`. This is a common requirement in many serialization libraries, but TypeScript does not provide built-in support for this feature.

Let’s expand the `TextShapeExtra` class to include a `Point` object:

```ts
class Point {
  constructor(
    public left: number,
    public right: number,
  ) {}
}

class TextShapeExtra {
  @SerialName("t")
  text: string;

  @SerialName("fs")
  fontSize: number;

  @SerialName("p")
  position: Point;
}
```

After serialization, the `TextShapeExtra` object should look like this:

```json
{
  "t": "Hello",
  "fs": 14,
  "p": "10|20"
}
```

To achieve this, we apply the same approach as we did for custom field names: create a decorator that allows us to specify custom serialization and deserialization logic for a field.

Sample implementation:

```ts
export function Serialize<T>(
  serializer: (value: T) => any,
  deserializer: (value: any) => T,
): PropertyDecorator {
  return function (
    target: Object,
    propertyKey: string | symbol,
  ): void {
    const constructor = target.constructor as JsonizableClass<any>;

    if (!constructor.serializers) {
      constructor.serializers = new Map();
    }
    if (!constructor.deserializers) {
      constructor.deserializers = new Map();
    }

    constructor.serializers.set(propertyKey.toString(), serializer);
    constructor.deserializers.set(propertyKey.toString(), deserializer);
  };
}
```

Sample usage:

```ts
class TextShapeExtra {
  @SerialName("t")
  text: string;

  @SerialName("fs")
  fontSize: number;

  @SerialName("p")
  @Serialize<Point>(
    (p) => `${p.left}|${p.right}`,
    (s) => {
      const [l, r] = s.split("|").map(Number);
      return new Point(l, r);
    },
  )
  position: Point;
}
```

Similar to the `SerialName` decorator, the `Serialize` decorator stores the serialization and deserialization logic in the `serializers` and `deserializers` properties of the class.

With this decorator, we can now serialize and deserialize objects with custom serialization and deserialization logic without having to write custom mapping logic for each class.

---

### Support for Nested Objects

Finally, we need to support nested objects in the serialization and deserialization process. This means that if an object contains another object, the nested object should also be serialized and deserialized according to the custom field names and serialization logic.

The text shape extra belongs to a text shape:

```ts
class TextShape {
  @SerialName("i")
  id: string;

  @SerialName("e")
  extra: TextShapeExtra;
}
```

After serialization, the `TextShape` object should look like this:

```json
{
  "i": "shape-1",
  "e": {
    "t": "Hello",
    "fs": 14,
    "p": "10|20"
  }
}
```

To handle this, we have two options:

1. Use the `Serialize` decorator to specify custom serialization and deserialization logic for nested objects.  
2. Recursively serialize and deserialize nested objects.

The first option is straightforward and allows us to customize the serialization and deserialization logic for nested objects. However, it requires us to manually specify the serialization and deserialization logic for each nested object, which can be tedious and error-prone due to repeated code.

The second option is more complex but offers a more generic solution. We can recursively serialize and deserialize nested objects by checking if a field is an object and applying the serialization and deserialization logic accordingly.

To do this, we need the 3rd decorator, but this time, not for the field, but for the class itself. The code is a bit long:

```ts
export function Jsonizable(): ClassDecorator {
  return function <T extends { new (...args: any[]): {} }>(
    constructor: T,
  ) {
    const jsonizableClass = constructor as JsonizableClass<any>;

    constructor.prototype.toJson = function (): any {
      const result: any = {};
      const serialNames = jsonizableClass.serialNames ?? new Map();
      const serializers = jsonizableClass.serializers ?? new Map();

      for (const key of Object.keys(this)) {
        const jsonKey = serialNames.get(key) ?? key;
        const value = (this as any)[key];

        if (serializers.has(key)) {
          result[jsonKey] = serializers.get(key)!(value);
        } else if (value && typeof value.toJson === "function") {
          result[jsonKey] = value.toJson();
        } else {
          result[jsonKey] = value;
        }
      }

      return result;
    };

    jsonizableClass.fromJson = function (json: any) {
      const instance = new constructor() as any;
      const serialNames = jsonizableClass.serialNames ?? new Map();
      const deserializers = jsonizableClass.deserializers ?? new Map();

      // Reverse map for JSON key -> property key
      const reverseNames = new Map<string, string>();
      for (const [prop, jsonKey] of serialNames.entries()) {
        reverseNames.set(jsonKey, prop);
      }

      for (const jsonKey of Object.keys(json)) {
        const propKey = reverseNames.get(jsonKey) ?? jsonKey;
        const value = json[jsonKey];

        if (deserializers.has(propKey)) {
          instance[propKey] = deserializers.get(propKey)!(value);
        } else {
          instance[propKey] = value;
        }
      }

      return instance;
    };

    return constructor;
  };
}
```

This decorator adds `toJson` and `fromJson` methods to the class and class’s static API respectively, which recursively serializes and deserializes the object and its nested objects.

- `toJson` method iterates over each field in the object, checks if the field has a custom serializer, and applies the serialization logic accordingly. If the field is an object, it recursively calls the `toJson` method of the nested object.
- `fromJson` method does the reverse, iterating over each field in the JSON data, checking if the field has a custom deserializer, and applying the deserialization logic accordingly. If the field is an object, it recursively calls the `fromJson` method of the nested object.

With this decorator, we can now serialize and deserialize objects with nested objects without having to write custom serialization and deserialization logic for each nested object.

> Note: This solution does not automatically support arrays or polymorphism. To handle these cases, we can use the `@Serialize` decorator to specify custom serialization and deserialization logic for arrays and polymorphic objects.

---

### Combining Everything Together

Definition side:

```ts
@Jsonizable()
class TextShapeExtra {
  @SerialName("t")
  text: string;

  @SerialName("fs")
  fontSize: number;

  @SerialName("p")
  @Serialize<Point>(
    (p) => `${p.left}|${p.right}`,
    (s) => {
      const [l, r] = s.split("|").map(Number);
      return new Point(l, r);
    },
  )
  position: Point;
}

@Jsonizable()
class TextShape {
  @SerialName("i")
  id: string;

  @SerialName("e")
  extra: TextShapeExtra;
}
```

Caller side:

```ts
const shape = new TextShape();
shape.id = "shape-1";
shape.extra = new TextShapeExtra();
shape.extra.text = "Hello";
shape.extra.fontSize = 14;
shape.extra.position = new Point(10, 20);

const json = shape.toJson();
// Serialize to string if needed:
const jsonString = JSON.stringify(json, null, 2);

const parsed = JSON.parse(jsonString);
const restoredShape = TextShape.fromJson!(parsed);
```

---

### Setting up the Project

One thing I haven’t mentioned is that decorators are not natively supported by TypeScript and are still considered an [experimental feature](https://www.typescriptlang.org/tsconfig/#experimentalDecorators). To enable their usage, the `experimentalDecorators` flag must be set to true in the `tsconfig.json` file. This configuration is similar to what [class-transformer](https://github.com/typestack/class-transformer/blob/a073b5ea218dd4da9325fe980f15c1538980500e/docs/pages/01-getting-started.md#installation) requires:

```jsonc
{
  "compilerOptions": {
    "target": "ES6",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": false,
    "strict": true
  }
}
```

---

### End Notes

In MonoSketch Kotlin version, except for [Compose HTML](https://github.com/JetBrains/compose-multiplatform?tab=readme-ov-file#compose-html) for UI, the only library is the serialization. Now, after implementing the custom solution in TypeScript, MonoSketch TypeScript version uses Svelte as the only 3rd party library. How crazy I am!