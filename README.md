# typescripttypes-1
Typescript types

```js
// Type Predicates

type FinalInvoice = {
    __typeName: "FinalInvoice"
}

type DraftInvoice = {
    __typeName: "DraftInvoice"
}

type Invoice = FinalInvoice | DraftInvoice;

const isFinalInvoice = (invoice: Invoice): invoice is FinalInvoice => {
    return invoice.__typeName === "FinalInvoice";
}

const isDraftInvoice = (invoice: Invoice): invoice is DraftInvoice => invoice.__typeName === "FinalInvoice";

// Asserts Functions

function assertIsNumber(val: any): asserts val is number {
    if (typeof val !== "number") {
        throw new Error("Not a Number");
    }
}

function double(input: any) {
    assertIsNumber(input)
    return input * 2;
}

// Lockup types

type Route = {
    origin: {
        city: string;
        state: string;
        depatureFee: number;
    },
    destination: {
        city: string;
        state: string;
        arrivalFee: number;
    }
}

type Origin = Route['origin'];
type Destination = Route['destination']


// Generics

enum TaskType {
    feature = "feature",
    bug = "bug"
}

type Task<T = TaskType> = {
    name: string;
    type: T
};

const whatever: Task = { name: "single sign on", type: TaskType.feature };
whatever.type = TaskType.bug;

type FeatureTask = Task<TaskType.feature>;
const featureTask: FeatureTask = { name: 'singlesing one', type: TaskType.feature };
// featureTask.type = TaskType.bug;

type BugTas = Task<TaskType.bug>;


// Extract Utility function

type Trip = {
    origin: {
        uuid: string;
        city: string;
        state: string;
    }
} | { originUuid: string }

type TripWithOriginalRef = Extract<Trip, { originUuid: string }>;

type TripWithOriginWhole = Extract<Trip, { origin: { uuid: string } }>;

const hasOriginRef = (trip: Trip): trip is TripWithOriginalRef => "originUuid" in trip;


// Conditional types

type Diesel = {
    type: "petrol" | "bio" | "synthetic";
}

type Gasoline = {
    type: "hybrid" | "conventional"
}

type Bus = {
    engine: Diesel
}

type Car = {
    engine: Gasoline
}

type Engine<T> = T extends { engine: unknown } ? T["engine"] : never;

type BusEngine = Engine<Bus>;

type CarEngine = Engine<Car>;

type Bicycle = {
    power: "limbs"
};

type NoEngine = Engine<Bicycle>;

// const noEngine: NoEngine = {type: "limbs"};


type Unarray<T> = T extends Array<infer U> ? U : T;
enum Priority {
    MustHave = "MustHave",
    CanHave = "CanHave"
}

const backlog = {
    releases: [
        {
            name: "Sprint 1",
            epics: [
                {
                    name: "Account Manager",
                    tasks: [
                        {
                            name: "Single Sign On", priority: Priority.MustHave
                        },
                        {
                            name: "Email Notifications", priority: Priority.CanHave
                        }
                    ]
                }
            ]
        }
    ],
    info: {
        week: 25,
        year: 2022,
        title: "Development and Deployment"
    }
}

type Release = Unarray<typeof backlog["releases"]>;

type Info = Unarray<typeof backlog["info"]>;


// Conditional Type
type SomeType = string;
type MyConditionalType = SomeType extends string ? string : never;


function someFunction<T>(value: T) {
    type A = T extends boolean ? 'TYPE A' : T extends string ? 'TYPE B' : never;

    return (someArg: T extends boolean ? 'TYPE A' : 'TYPE B') => {
        const a: 'TYPE A' | 'TYPE B' = someArg;
    }
}

const result = someFunction<string>('Madhu');

type StringOrNot<T> = T extends string ? string : never;

type AUnion = string | boolean | never;

type ResultType = Exclude<'a' | 'b', 'c'>;

type MayType<T> = [T] extends [string | number] ? T : never;

type MyResult = MayType<boolean | string>;


type Infersomething<T> = T extends infer U ? U : never;

type InferedT = Infersomething<'I am a string'>;

type InferSomething2<T> = T extends { a: infer A, b: number } ? A : any;

type Infered2 = InferSomething2<{ a: 'Hello', b: 1 }>;

// don't use enum
enum LogLevel {
    DEBUG,
    WARNING,
    ERROR
}

const AuthMethod = {
    push: "Push",
    sms: "Sms",
    voice: "Voice"
} as const;

type AuthMethod = typeof AuthMethod[keyof typeof AuthMethod];

function doSome(input: AuthMethod) {
    console.log(input);
}

doSome("Push")

type User = Record<string, number | string>;

const user = {
    name: "Madhu",
    age: 35
} satisfies User;

let myName = user.name;


type Prop<T> = {
    get: () => T;
    set: (t: T) => void;
}

function createProp<T>(val: T): Prop<T> {
    return {
        get() { return val },
        set(v: T) { val = v }
    }
}
const props = {
    name: createProp("liz"),
    age: createProp(10)
}

type Props = typeof props;
type Key = keyof Props;


type None = { _type: 'none' }
type Some<T> = { _type: 'some', value: T }

type Option<T> = None | Some<T>;

const none: None = { _type: 'none' }
const some = <T>(value: T): Some<T> => ({ _type: 'some', value });

// 1- Create
function optionalCatch<T>(fn: () => T): Option<T> {
    try {
        return some(fn())
    } catch (err) {
        return none;
    }
}


function greet(name: string) {
    if (name === 'andrew') {
        throw new Error('Dont ');
    }
    return `Hello ${name}`
}


const maybeGreeting = optionalCatch(() => greet('andrew'));


async function optionalResolve<T>(p: Promise<T>): Promise<Option<T>> {
    // try {
    //     return some(await p);
    // } catch(err){
    //     return none;
    // }
    return p.then(v => some(v)).catch(() => none);
}



const maybeCount = optionalResolve(Promise.resolve(32));

function toOptional<I, O extends I>(fn: (i: I) => i is O) {
    return function (arg: I): Option<O> {
        try {
            return fn(arg) ? some(arg) : none
        } catch (err) {
            return none;
        }
    }
}

const optionalDefined = toOptional(<T>(arg: T | undefined | null): arg is T => arg != null);

const array = [1, 2]
const b = optionalDefined(array.pop());


// Consume
function unwrap<T>(option: Option<T>): T {
    if (option._type === 'some') {
        return option.value
    }
    throw new Error("Could not unwrap option");
}

function unwrapOr<T>(option: Option<T>, fallback: T): T {
    try {
        return unwrap(option);
    } catch (err) {
        return fallback;
    }
}

function unwrapExpect<T>(opt: Option<T>, message: string): T {
    if (opt._type === 'some') return opt.value;
    throw new Error(message);
}

function getTimeDiff(arr: Date[]): number {
    const start = optionalDefined(arr[0]);
    const end = optionalDefined(arr[1]);

    const startVal = unwrapExpect(start, 'range must have start date').valueOf();
    const endVal = unwrapOr(end, new Date()).valueOf();
    return endVal - startVal;
}
```
