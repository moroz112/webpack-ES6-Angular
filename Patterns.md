<h1>Patterns</h1>
<div>
    <strong>Singleton:</strong></br>
    <div>Паттерн проектирования, который гарантирует что для определенного класса будет создан только один екзкмпляр. Предоставляет глобальную тчку доступа к этому екземпляру</div>
    var Singleton = (function () {
        var instance;

        function createInstance() {
            var object = new Object("I am the instance");
            return object;
        }

        return {
            getInstance: function () {
                if (!instance) {
                    instance = createInstance();
                }
                return instance;
            }
        };
    })();

    function run() {

        var instance1 = Singleton.getInstance();
        var instance2 = Singleton.getInstance();

        alert("Same instance? " + (instance1 === instance2));
    }
    <strong>Factory:</strong></br>
    <div>Паттерн проектирования при котором глобально создается только интерфейс. Создание инстанса фабрика делегирует своим подклассам</div>
    function Factory() {
        this.createEmployee = function (type) {
            var employee;

            if (type === "fulltime") {
                employee = new FullTime();
            } else if (type === "parttime") {
                employee = new PartTime();
            } else if (type === "temporary") {
                employee = new Temporary();
            } else if (type === "contractor") {
                employee = new Contractor();
            }

            employee.type = type;

            employee.say = function () {
                log.add(this.type + ": rate " + this.hourly + "/hour");
            }

            return employee;
        }
    }

    var FullTime = function () {
        this.hourly = "$12";
    };

    var PartTime = function () {
        this.hourly = "$11";
    };

    var Temporary = function () {
        this.hourly = "$10";
    };

    var Contractor = function () {
        this.hourly = "$15";
    };


    function run() {
        var employees = [];
        var factory = new Factory();

        employees.push(factory.createEmployee("fulltime"));
        employees.push(factory.createEmployee("parttime"));
        employees.push(factory.createEmployee("temporary"));
        employees.push(factory.createEmployee("contractor"));

        for (var i = 0, len = employees.length; i < len; i++) {
            employees[i].say();
        }
    }
    <strong>Observer:</strong></br>
    <div>Паттерн проектирования при котором обьекты подписываются на ивенты и уведомляются тогда когда эти ивенты выполняются</div>

    function Click() {
        this.handlers = [];  // observers
    }

    Click.prototype = {

        subscribe: function(fn) {
            this.handlers.push(fn);
        },

        unsubscribe: function(fn) {
            this.handlers = this.handlers.filter(
                function(item) {
                    if (item !== fn) {
                        return item;
                    }
                }
            );
        },

        fire: function(o, thisObj) {
            var scope = thisObj || window;
            this.handlers.forEach(function(item) {
                item.call(scope, o);
            });
        }
    }

    // log helper

    function run() {

        var clickHandler = function(item) {
            log.add("fired: " + item);
        };

        var click = new Click();

        click.subscribe(clickHandler);
        click.fire('event #1');
        click.unsubscribe(clickHandler);
        click.fire('event #2');
        click.subscribe(clickHandler);
        click.fire('event #3');
    }
    <strong>Decorator:</strong></br>
    <div>Паттерн проектирования при использовании которого происходит динамическая привязка функциональности к обьекту</div>

    var User = function(name) {
        this.name = name;

        this.say = function() {
            log.add("User: " + this.name);
        };
    }

    var DecoratedUser = function(user, street, city) {
        this.user = user;
        this.name = user.name;  // ensures interface stays the same
        this.street = street;
        this.city = city;

        this.say = function() {
            log.add("Decorated User: " + this.name + ", " +
                       this.street + ", " + this.city);
        };
    }

    function run() {

        var user = new User("Kelly");
        user.say();

        var decorated = new DecoratedUser(user, "Broadway", "New York");
        decorated.say();
    }
    <strong>Facade:</strong>
    <div>Паттерн который предоставляет интерфейс за которым скрывается сложная функциональность одной или нескольких подсистем</div>
    var Mortgage = function(name) {
        this.name = name;
    }

    Mortgage.prototype = {

        applyFor: function(amount) {
            // access multiple subsystems...
            var result = "approved";
            if (!new Bank().verify(this.name, amount)) {
                result = "denied";
            } else if (!new Credit().get(this.name)) {
                result = "denied";
            } else if (!new Background().check(this.name)) {
                result = "denied";
            }
            return this.name + " has been " + result +
                   " for a " + amount + " mortgage";
        }
    }

    var Bank = function() {
        this.verify = function(name, amount) {
            // complex logic ...
            return true;
        }
    }

    var Credit = function() {
        this.get = function(name) {
            // complex logic ...
            return true;
        }
    }

    var Background = function() {
        this.check = function(name) {
            // complex logic ...
            return true;
        }
    }

    function run() {
        var mortgage = new Mortgage("Joan Templeton");
        var result = mortgage.applyFor("$100,000");

        alert(result);
    }
    <strong>Mediator:</strong></br>
    <div>Паттерн который определяет обьект что инкапсулирует в себе логику взаимодействия других обьектов, обеспечивая слабую связь между ними</div>
    var Participant = function(name) {
        this.name = name;
        this.chatroom = null;
    };

    Participant.prototype = {
        send: function(message, to) {
            this.chatroom.send(message, this, to);
        },
        receive: function(message, from) {
            log.add(from.name + " to " + this.name + ": " + message);
        }
    };

    var Chatroom = function() {
        var participants = {};

        return {

            register: function(participant) {
                participants[participant.name] = participant;
                participant.chatroom = this;
            },

            send: function(message, from, to) {
                if (to) {                      // single message
                    to.receive(message, from);
                } else {                       // broadcast message
                    for (key in participants) {
                        if (participants[key] !== from) {
                            participants[key].receive(message, from);
                        }
                    }
                }
            }
        };
    };

    function run() {
        var yoko = new Participant("Yoko");
        var john = new Participant("John");
        var paul = new Participant("Paul");
        var ringo = new Participant("Ringo");

        var chatroom = new Chatroom();
        chatroom.register(yoko);
        chatroom.register(john);
        chatroom.register(paul);
        chatroom.register(ringo);

        yoko.send("All you need is love.");
        yoko.send("I love you John.");
        john.send("Hey, no need to broadcast", yoko);
        paul.send("Ha, I heard that!");
        ringo.send("Paul, what do you think?", paul);
    }
    <strong>Adapter:</strong>
    <div>Конвертирует интерфейс класса в интерфейс, который ожидает клиент</div>
    function Shipping() {
        this.request = function(zipStart, zipEnd, weight) {
            // ...
            return "$49.75";
        }
    }

    // new interface

    function AdvancedShipping() {
        this.login = function(credentials) { /* ... */ };
        this.setStart = function(start) { /* ... */ };
        this.setDestination = function(destination) { /* ... */ };
        this.calculate = function(weight) { return "$39.50"; };
    }

    // adapter interface

    function ShippingAdapter(credentials) {
        var shipping = new AdvancedShipping();

        shipping.login(credentials);

        return {
            request: function(zipStart, zipEnd, weight) {
                shipping.setStart(zipStart);
                shipping.setDestination(zipEnd);
                return shipping.calculate(weight);
            }
        };
    }

    function run() {
        var shipping = new Shipping();
        var credentials = {token: "30a8-6ee1"};
        var adapter = new ShippingAdapter(credentials);

        // original shipping object and interface

        var cost = shipping.request("78701", "10010", "2 lbs");
        log.add("Old cost: " + cost);

        // new shipping object with adapted interface

        cost = adapter.request("78701", "10010", "2 lbs");
    }
    <strong>Proxy:</strong>
    <div>Паттерн, который предоставляет обьект, который контролирут доступ к другому обьекту, перехватывая все вызовы</div>
    function GeoCoder() {

        this.getLatLng = function(address) {

            if (address === "Amsterdam") {
                return "52.3700° N, 4.8900° E";
            } else if (address === "London") {
                return "51.5171° N, 0.1062° W";
            } else if (address === "Paris") {
                return "48.8742° N, 2.3470° E";
            } else if (address === "Berlin") {
                return "52.5233° N, 13.4127° E";
            } else {
                return "";
            }
        };
    }

    function GeoProxy() {
        var geocoder = new GeoCoder();
        var geocache = {};

        return {
            getLatLng: function(address) {
                if (!geocache[address]) {
                    geocache[address] = geocoder.getLatLng(address);
                }
                log.add(address + ": " + geocache[address]);
                return geocache[address];
            },
            getCount: function() {
                var count = 0;
                for (var code in geocache) { count++; }
                return count;
            }
        };
    };

    function run() {
        var geo = new GeoProxy();

        // geolocation requests

        geo.getLatLng("Paris");
        geo.getLatLng("London");
        geo.getLatLng("London");
        geo.getLatLng("London");
        geo.getLatLng("London");
        geo.getLatLng("Amsterdam");
        geo.getLatLng("Amsterdam");
        geo.getLatLng("Amsterdam");
        geo.getLatLng("Amsterdam");
        geo.getLatLng("London");
        geo.getLatLng("London");
    }
</div>