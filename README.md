# typescripttypes-1
Typescript types

`
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
    if(typeof val !== "number") {
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
    destination:{
        city: string;
        state: string;
        arrivalFee: number;
    }
}

type Origin = Route['origin']
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

const whatever: Task ={name: "single sign on", type: TaskType.feature};
whatever.type = TaskType.bug;

type FeatureTask = Task<TaskType.feature>;
const featureTask: FeatureTask = {name: 'singlesing one', type: TaskType.feature};
// featureTask.type = TaskType.bug;

type BugTas = Task<TaskType.bug>;


// Extract Utility function

type Trip = {origin: {
    uuid: string;
    city: string;
    state: string;
}} | {originUuid: string}

type TripWithOriginalRef = Extract<Trip, {originUuid: string}>;

type TripWithOriginWhole = Extract<Trip, {origin: {uuid: string}}>;

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

type Engine<T> = T extends {engine: unknown} ? T["engine"]: never;

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

`
