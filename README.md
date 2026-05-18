# Assignment2OOP
# PakWheels Clone — Car Marketplace Management System
### C++ Object-Oriented Programming — Assignment 2

**Student Name:** Syed Muhammad Yahya Raza
**Roll Number:** 25K-0019
**Class:** BAI-2C

---

## Table of Contents

1. [Abstract Interfaces](#1-abstract-interfaces)
2. [Engine Class](#2-engine-class)
3. [Vehicle Listing](#3-vehicle-listing)
4. [Car & Bike — Polymorphism](#4-car--bike--polymorphism)
5. [User, Seller & Admin](#5-user-seller--admin)
6. [Buyer Accounts](#6-buyer-accounts)
7. [Search & Filter Listings](#7-search--filter-listings)
8. [Operator Overloading](#8-operator-overloading)
9. [Friend Functions](#9-friend-functions)
10. [Admin Panel](#10-admin-panel)

---

## 1. Abstract Interfaces

The system defines four abstract interfaces that enforce contracts across all major classes. `ISearchable` enables keyword-based listing search. `IReportable` requires any user role that generates reports (Seller, Admin) to implement `genReport()` and `getSummaryTxt()`. `IPriceable` separates pricing logic from the Listing class. `IVerifiable` enforces a standard way to check and set verification state on Seller accounts.

### Screenshot

![Abstract Interfaces](./Images/image1.png)

### Code

```cpp
class ISearchable {
public:
    virtual bool   matchesQuery(const string& kw)  const = 0;
    virtual string getSearchKey()                  const = 0;
    virtual ~ISearchable() {}
};

class IReportable {
public:
    virtual void   genReport()     const = 0;
    virtual string getSummaryTxt() const = 0;
    virtual ~IReportable() {}
};

class IPriceable {
public:
    virtual double  getBasePrice()              const = 0;
    virtual double  getDiscountedPrice(int pct) const = 0;
    virtual ~IPriceable() {}
};

class IVerifiable {
public:
    virtual bool isVerified() const = 0;
    virtual void doVerify()         = 0;
    virtual ~IVerifiable() {}
};
```

---

## 2. Engine Class

`Engine` is a standalone composition class embedded inside `Vehicle`. It stores engine kind, CC displacement, horsepower, torque, and turbo flag. It overloads `==` to compare two engines by type and CC, and overloads `+` to merge two engine specs into a combined object. A friend function `printEngineSecret()` accesses private members directly for admin diagnostic dumps.

### Screenshot

![Engine Class](./Images/image2.png)

### Code

```cpp
class Engine {
private:
    string engKind;
    float  ccSize;
    int    horsePwr;
    int    torque;
    bool   hasTurbo;
public:
    bool operator==(const Engine& e) const {
        return (engKind == e.engKind && (int)ccSize == (int)e.ccSize);
    }
    Engine operator+(const Engine& e) const {
        Engine merged;
        merged.engKind  = engKind + "+" + e.engKind;
        merged.ccSize   = ccSize  + e.ccSize;
        merged.horsePwr = horsePwr + e.horsePwr;
        merged.torque   = torque   + e.torque;
        merged.hasTurbo = hasTurbo || e.hasTurbo;
        return merged;
    }
    friend void printEngineSecret(const Engine& e);
};
```

---

## 3. Vehicle Listing

The `Listing` class inherits both `ISearchable` and `IPriceable`. Each listing wraps a `Vehicle` pointer, a seller ID, price, location, and approval/active flags. It implements `matchesQuery()` for keyword-based search across brand, model, and location, `getBasePrice()` and `getDiscountedPrice()` for pricing, and overloads `operator<` to compare two listings by price. A friend function `comparePrices()` accesses the private price field of both listings directly for admin audit reports.

### Screenshot

![Vehicle Listing](./Images/image3.png)

### Code

```cpp
class Listing : public ISearchable, public IPriceable {
private:
    int      listingID;
    Vehicle* vehicle;
    int      sellerID;
    double   price;
    string   location;
    bool     isApproved;
    bool     isActive;
    static const string CURRENCY;
    static int listCnt;
    static int nextLID;
public:
    bool matchesQuery(const string& kw) const override {
        if (!vehicle) return false;
        return (vehicle->getBrand().find(kw) != string::npos ||
                vehicle->getModel().find(kw) != string::npos ||
                location.find(kw)            != string::npos);
    }
    double getBasePrice() const override { return price; }
    double getDiscountedPrice(int pct) const override {
        if (pct < 0 || pct > 100) return price;
        return price - (price * pct / 100.0);
    }
    bool operator<(const Listing& l) const { return price < l.price; }
    friend void comparePrices(const Listing& a, const Listing& b);
};
```

---

## 4. Car & Bike — Polymorphism

`Car` and `Bike` both inherit from the abstract `Vehicle` class and provide their own override of `display()` and `getVehicleType()`. `Vehicle` declares these as pure virtual, enforcing runtime polymorphism. `Car` adds door count, gearbox type, drive mode, fuel efficiency, and AC flag. `Bike` adds bike type, fairing panel, gear count, seat height, and sidecar flag.

### Screenshot

![Car and Bike](./Images/image4.png)

### Code

```cpp
class Car : public Vehicle {
private:
    int    doorCnt;
    string gearboxType;
    string driveMode;
    float  kmPerLtr;
    bool   hasAC;
    static const string SAFETY_STANDARD;
public:
    void display() const override { ... }
    string getVehicleType() const override { return "Car"; }
};

class Bike : public Vehicle {
private:
    string bikeType;
    bool   hasFairPanel;
    int    gearCnt;
    float  seatHtMM;
    bool   hasSideCar;
    static const string BIKE_STANDARD;
public:
    void display() const override { ... }
    string getVehicleType() const override { return "Bike"; }
};
```

---

## 5. User, Seller & Admin

`User` is an abstract base class with static `userCount` and `nextUID` members. It overloads `operator==` to compare accounts by UID. `Seller` inherits `User`, `IReportable`, and `IVerifiable` — implementing `genReport()`, `getSummaryTxt()`, `isVerified()`, and `doVerify()`. `Admin` inherits `User` and `IReportable` and can approve or remove listings and ban users.

### Screenshot

![Seller and Admin](./Images/image5.png)

### Code

```cpp
class Seller : public User, public IReportable, public IVerifiable {
private:
    string   bizName;
    float    rating;
    int      salesDone;
    bool     verified;
public:
    void genReport()  const override { ... }
    bool isVerified() const override { return verified; }
    void doVerify()         override { verified = true; ... }
    friend bool isCompatible(const Seller& s, const Buyer& b);
};

class Admin : public User, public IReportable {
public:
    bool okayListing(Listing& listing) { ... }
    bool pullListing(Listing& listing)  { ... }
    void blockUser  (User& target)      { ... }
};
```

---

## 6. Buyer Accounts

`Buyer` inherits from `User` and stores a budget, preferred brand, a favorites array of `Listing` pointers, and an array of sent `Message` objects. Buyers can save listings to favorites, view saved listings, send messages to sellers, and view sent messages. The friend function `transferBudget()` accesses the private budget field of two `Buyer` objects directly to move funds between them.

### Screenshot

![Buyer Accounts](./Images/image6.png)

### Code

```cpp
class Buyer : public User {
private:
    float    budget;
    string   favBrand;
    Listing* favorites[MAX_FAVORITES];
    int      favCount;
    Message  sentMessages[50];
    int      msgCount;
public:
    bool addToFavs(Listing* listing) { ... }
    void showFavs()                  const { ... }
    bool sendMsg(int receiverID, const string& content,
                 const string& ts) { ... }
    void showSentMsgs()              const { ... }
    friend void transferBudget(Buyer& giver, Buyer& recvr, float amt);
};
```

---

## 7. Search & Filter Listings

The `filterListings()` function offers six filter modes: by brand, model, price range, year, maximum mileage, and keyword. The keyword search uses the `ISearchable` interface polymorphically — each `Listing` is upcast to `ISearchable*` and `matchesQuery()` is called virtually.

### Screenshot

![Filter Listings](./Images/image7.png)

### Code

```cpp
void filterListings() {

    // keyword search — uses ISearchable polymorphically
    string kw; cout << "  Keyword: "; getline(cin, kw);
    for (int i = 0; i < lstCount; i++) {
        ISearchable* srch = listings[i];
        if (srch && listings[i]->isActiveStatus()
                 && srch->matchesQuery(kw)) {
            listings[i]->printCompact();
        }
    }
}
```

---

## 8. Operator Overloading

Five operators are overloaded across three classes. `Engine` overloads `==` (compare by type and CC) and `+` (merge two engines). `Vehicle` overloads `==` (compare by brand, model, year) to detect duplicate listings. `User` overloads `==` (compare by UID) for duplicate account detection. `Listing` overloads `<` (compare by price) for sorting and best-deal queries.

### Screenshot

![Operator Overloading](./Images/image8.png)

### Code

```cpp
// Engine : == and +
bool   operator==(const Engine& e) const { ... }  // same type + CC
Engine operator+ (const Engine& e) const { ... }  // merged spec

// Vehicle : ==
bool operator==(const Vehicle& v) const {
    return (brand == v.brand && model == v.model && year == v.year);
}

// User : ==
bool operator==(const User& u) const { return uid == u.uid; }

// Listing : <
bool operator<(const Listing& l) const { return price < l.price; }
```

---

## 9. Friend Functions

Four friend functions are defined. `printEngineSecret()` accesses `Engine` private members for diagnostic output. `transferBudget()` reads and writes `Buyer::budget` directly without getters. `comparePrices()` accesses `Listing::price` directly for an admin audit comparison. `isCompatible()` reads `Seller::rating` and `Buyer::budget` to determine buyer-seller compatibility.

### Screenshot

![Friend Functions](./Images/image9.png)

### Code

```cpp
void printEngineSecret(const Engine& e) {
    cout << e.engKind << "  " << e.horsePwr << "  " << e.hasTurbo;
}

void transferBudget(Buyer& giver, Buyer& recvr, float amt) {
    giver.budget -= amt;
    recvr.budget += amt;
}

void comparePrices(const Listing& a, const Listing& b) {
    if (a.price < b.price)
        cout << "Listing #" << a.listingID << " is cheaper.";
}

bool isCompatible(const Seller& s, const Buyer& b) {
    return (s.rating >= 3.5f) && (b.budget > 500000.0f);
}
```

---

## 10. Admin Panel

The Admin Panel provides seven operations: approve a listing, remove a listing, view all listings in detail, verify a seller (calls `IVerifiable::doVerify()`), view system statistics (static member counts across all classes), compare two listing prices using the `comparePrices` friend function, and generate a seller report using the `IReportable` interface through an upcast pointer.

### Screenshot

![Admin Panel](./Images/image10.png)

### Code

```cpp
// System statistics — static members across all classes
cout << "Total Users    : " << User::getUserCount()       << "\n";
cout << "Total Vehicles : " << Vehicle::getVehicleCount() << "\n";
cout << "Total Listings : " << Listing::getListCnt()      << "\n";
cout << "Total Messages : " << Message::getMsgCount()     << "\n";

// IReportable upcast — polymorphic report generation
IReportable* rpt = &sellers[si];
rpt->genReport();
```
