# Patterns

## 📑 Оглавление
- Диаграмма соответствий
- Порождающие
  - [Singleton](#1-singleton)
  - [Factory Method](#2-factory-method)
  - [Abstract Factory](#3-abstract-factory)
  - [Builder](#4-builder)
  - [Prototype](#5-prototype)
- Структурные
  - [Adapter](#6-adapter)
  - [Facade](#7-facade)
  - [Decorator](#8-decorator)
  - [Composite](#9-composite)
  - [Proxy](#10-proxy)
- Поведенческие
  - [Strategy](#11-strategy)
  - [Observer](#12-observer)
  - [Command](#13-command)
  - [State](#14-state)
  - [Chain of Responsibility](#15-chain-of-responsibility)

---

## Диаграмма соответствий

| Категория         | Паттерн                 | Когда применять (симптом)                                                     |
| ----------------- | ----------------------- | ----------------------------------------------------------------------------- |
| **Порождающие**   | Singleton               | Нужен единственный объект, глобальная точка доступа, ленивая инициализация    |
|                   | Factory Method          | Много подклассов одного семейства; выбор реализации отложен в подкласс        |
|                   | Abstract Factory        | Согласованные семейства продуктов (например, GUI под разные ОС)               |
|                   | Builder                 | Сложная пошаговая сборка объектов; нужны разные представления одного продукта |
|                   | Prototype               | Частое клонирование похожих объектов, дорого создавать «с нуля»               |
| **Структурные**   | Adapter                 | Подружить несовместимые интерфейсы («обёртка»)                                |
|                   | Facade                  | Спрятать сложность подсистемы за простой API                                  |
|                   | Decorator               | Динамически добавлять поведение без наследования                              |
|                   | Composite               | Иерархия «часть-целое»: дерево, где лист и узел обрабатываются одинаково      |
|                   | Proxy                   | Контроль доступа, кэш, ленивая загрузка, удалённый доступ                     |
| **Поведенческие** | Strategy                | Семейство алгоритмов, возможность менять во время исполнения                  |
|                   | Observer                | Подписка/уведомление множества объектов об изменениях                         |
|                   | Command                 | Инкапсуляция запроса как объекта; отмена/история                              |
|                   | State                   | Разные состояния объекта → разные поведения                                   |
|                   | Chain of Responsibility | Гибкая передача запроса по цепочке обработчиков                               |

---
# Порождающие

## 1) Singleton
**Проблема:** Нужен единственный объект (напр. конфиг/реестр/кеш) и единая точка доступа.  
**Симптом:** Один и тот же объект создаётся в разных местах, рассылается по всему коду через параметры/глобали.

**Плохой пример (симптом в коде):**
```ts
// Везде new Config(), состояния рассинхронизируются  
class Config {  
  constructor(public env = process.env.NODE_ENV ?? 'prod') {  
  }  
}  
  
const a = new Config();  
const b = new Config(); // другое место  
console.log(a.env === b.env); // может совпасть случайно, но это разные экземпляры
````

**Решение:** Приватный конструктор + статический геттер, лениво создающий единственный экземпляр.

**Код реализации:**

```ts
class Config {  
  private static instance?: Config;  
  
  private constructor(public readonly env = process.env.NODE_ENV ?? 'prod') {}  
  
  static getInstance(): Config {  
    if(!Config.instance){
	    Config.instance = new Config()
    };  
    
    return Config.instance;
  }  
}  
  
const cfg = Config.getInstance();
```

---

## 2) Factory Method

**Проблема:** Тип создаваемого объекта заранее неизвестен, но у него есть общий интерфейс.  
**Симптом:** Большие `switch/if` по типу при создании.

**Плохой пример (симптом в коде):**

```ts
type PayType = 'card' | 'cash' | 'crypto';  
  
function createPayment(type: PayType) {  
  if (type === 'card')  
    return {  
      pay: (a: number) => console.log('card', a)  
    };  
  if (type === 'cash')  
    return {  
      pay: (a: number) => console.log('cash', a)  
    };  
  if (type === 'crypto')  
    return {  
      pay: (a: number) => console.log('crypto', a)  
    };  
  throw new Error('Unknown');  
}
```

**Решение:** Делегировать создание подклассам/классам-создателям.

**Код реализации:**

```ts
abstract class Payment {  
  abstract pay(amount: number): void;  
}  
  
class CardPayment extends Payment {  
  pay(a: number) {  
    console.log(`Карта: ${a}₽`);  
  }  
}  
  
class CashPayment extends Payment {  
  pay(a: number) {  
    console.log(`Наличные: ${a}₽`);  
  }  
}  
  
abstract class PaymentCreator {  
  abstract create(): Payment;  
}  
  
class CardPaymentCreator extends PaymentCreator {  
  create() {  
    return new CardPayment();  
  }  
}  
  
const p = new CardPaymentCreator().create();  
p.pay(1999);
```

---

## 3) Abstract Factory

**Проблема:** Нужны «согласованные» семейства продуктов (напр., виджеты GUI под Win/Mac).  
**Симптом:** В коде много `if (platform === 'win')` вокруг каждого компонента.

**Плохой пример (симптом в коде):**

```ts
function renderUI(platform: 'win' | 'mac') {  
  const btn = platform === 'win' ? 'WinButton' : 'MacButton';  
  const chk = platform === 'win' ? 'WinCheckbox' : 'MacCheckbox';  
  // дублирование логики выбора по платформе везде  
}
```

**Решение:** Ввести фабрику семейства и выбирать фабрику один раз.

**Код реализации:**

```ts
interface Button {  
  render(): string;  
}  
  
interface Checkbox {  
  check(): string;  
}  
  
class WinButton implements Button {  
  render() {  
    return 'WinButton';  
  }  
}  
  
class MacButton implements Button {  
  render() {  
    return 'MacButton';  
  }  
}  
  
class WinCheckbox implements Checkbox {  
  check() {  
    return 'WinCheckbox';  
  }  
}  
  
class MacCheckbox implements Checkbox {  
  check() {  
    return 'MacCheckbox';  
  }  
}  
  
interface GUIFactory {  
  createButton(): Button;  
  
  createCheckbox(): Checkbox;  
}  
  
class WinFactory implements GUIFactory {  
  createButton() {  
    return new WinButton();  
  }  
  
  createCheckbox() {  
    return new WinCheckbox();  
  }  
}  
  
class MacFactory implements GUIFactory {  
  createButton() {  
    return new MacButton();  
  }  
  
  createCheckbox() {  
    return new MacCheckbox();  
  }  
}  
  
function buildUI(factory: GUIFactory) {  
  const btn = factory.createButton();  
  const chk = factory.createCheckbox();  
  console.log(btn.render(), chk.check());  
}  
  
buildUI(new WinFactory());
```

---

## 4) Builder

**Проблема:** Объект сложный (много полей, опций), конструктор разрастается.  
**Симптом:** Конструкторы с 10+ параметрами, множество `undefined/null`.

**Плохой пример (симптом в коде):**

```ts
// трудно читать и поддерживать
const report = new Report("Sales", true, undefined, undefined, "RU", 20, null, "A4", /* ... */);
```

**Решение:** Пошаговая сборка через цепочку вызовов.

**Код реализации:**

```ts
class Query {  
  constructor(  
    public select = '*',  
    public from?: string,  
    public where?: string  
  ) {}  
  
  toString() {  
    return `SELECT ${this.select}  
            FROM ${this.from} ${this.where ? ` WHERE ${this.where}` : ''}`;  
  }  
}  
  
class QueryBuilder {  
  private q = new Query();  
  
  select(cols: string) {  
    this.q.select = cols;  
    return this;  
  }  
  
  from(tbl: string) {  
    this.q.from = tbl;  
    return this;  
  }  
  
  where(cond: string) {  
    this.q.where = cond;  
    return this;  
  }  
  
  build() {  
    return this.q;  
  }  
}  
  
const sql = new QueryBuilder()
  .from('users')  
  .where('active=1')  
  .build();
```

---

## 5) Prototype

**Проблема:** Часто нужно создавать копии уже настроенных объектов.  
**Симптом:** Ручное копирование полей при каждом создании.

**Плохой пример (симптом в коде):**

```ts
const baseTpl = { title: 'Счёт', vat: 20, footer: 'Спасибо' };  

function cloneInvoiceTpl() {  
  return {  
    title: baseTpl.title,  
    vat: baseTpl.vat,  
    footer: baseTpl.footer  
  }; 
}
```

**Решение:** Объект умеет сам себя клонировать.

**Код реализации:**

```ts
interface Clonable<T> {  
  clone(): T;  
}  
  
class InvoiceTemplate implements Clonable<InvoiceTemplate> {  
  constructor(  
    public title: string,  
    public vat: number,  
    public footer: string,
    public subTemplate?: InvoiceTemplate
  ) {}  
  
  clone() {  
    return new InvoiceTemplate(
	    this.title, 
		this.vat,
		this.footer, 
		this.subTemplate ? subTemplate.clone() : undefined
	);  
  }  
}  
  
const tpl = new InvoiceTemplate('Счёт', 20, 'Спасибо');  
const tpl2 = tpl.clone();
```

---

# Структурные

## 6) Adapter

**Проблема:** Внешний сервис имеет «неудобный» API.  
**Симптом:** В проекте везде мелкие конвертации к этому API.

**Плохой пример (симптом в коде):**

```ts
// Клиент знает детали старого API  
class OldBankAPI {  
  sendMoney(sum: number) {  
    /* ... */  
  }  
}  
  
function pay(amount: number, bank: OldBankAPI) {  
  bank.sendMoney(amount); // связали клиент со старым API  
}
```

**Решение:** Ввести целевой интерфейс и адаптер к старому API.

**Код реализации:**

```ts
class CardSystem {  
  sendMoney(sum: number) {  
    console.log('CardSystem bank:', sum);  
  }  
}  

class CryptoSystem {  
  sendCripto(hash: number) {  
    console.log('CryptoSystem bank:', hash);  
  }  
} 

class CardCryptoSystemAdapter extends CardSystem {  
  constructor(private readonly system: CryptoSystem)
  sendMoney(sum: number) {  
	const hash = btoa(sum);
    this.system.sendCripto(hash)
  }  
} 
  
class Bank{  
  constructor(private system: CardSystem) {}  
  
  pay(amount: number) {  
    this.system.sendMoney(amount);  
  }  
}  
```

---

## 7) Facade

**Проблема:** Клиенту приходится знать много деталей подсистемы.  
**Симптом:** Много вызовов низкоуровневых методов из клиента.

**Плохой пример (симптом в коде):**

```ts
const cpu = new CPU();  
const mem = new Memory();  
const disk = new Disk();  
const data = disk.read();  
mem.load(0, data);  
cpu.start(); // клиент управляет всем процессом
```

**Решение:** Спрятать сложность за простой API фасада.

**Код реализации:**

```ts
class CPU {  
  start() {  
    console.log('CPU start');  
  }  
}  
  
class Memory {  
  load(addr: number, data: string) {  
    console.log('load', addr, data);  
  }  
}  
  
class Disk {  
  read() {  
    return 'OS';  
  }  
}  
  
class ComputerFacade {  
  constructor(  
    private cpu = new CPU(),  
    private mem = new Memory(),  
    private disk = new Disk()  
  ) {}  
  
  boot() {  
    this.mem.load(0, this.disk.read());  
    this.cpu.start();  
  }  
}  
  
new ComputerFacade().boot();
```

---

## 8) Decorator

**Проблема:** Нужно динамически добавлять фичи объекту.  
**Симптом:** Множество подклассов «КомбинацияА+Логирование+Кеш+Аудит».

**Плохой пример (симптом в коде):**

```ts
class OrderServiceWithLoggingAndCachingAndMetrics { /* ... монолит ... */ }
```

**Решение:** Оборачивать объект в один или несколько декораторов.

**Код реализации:**

```ts
interface Notifier {  
  send(msg: string): void;  
}  
  
class BaseNotifier implements Notifier {  
  send(msg: string) {  
    console.log('Base:', msg);  
  }  
}  
  
class SMSDecorator implements Notifier {  
  constructor(private wrappee: Notifier) {}  
  
  send(msg: string) {  
    this.wrappee.send(msg);  
    console.log('SMS:', msg);  
  }  
}  
  
class EmailDecorator implements Notifier {  
  constructor(private wrappee: Notifier) {}  
  
  send(msg: string) {  
    this.wrappee.send(msg);  
    console.log('Email:', msg);  
  }  
}  
  
const notifier = new EmailDecorator(new SMSDecorator(new BaseNotifier()));  
notifier.send('Поступила заявка');
```

---

## 9) Composite

**Проблема:** Нужно одинаково обходить дерево (папки/файлы, меню/пункты).  
**Симптом:** В коде везде `if (leaf) else (node)`.

**Плохой пример (симптом в коде):**

```ts
type Node = { name: string; children?: Node[] };  
  
function print(n: Node) {  
  if (!n.children) console.log(n.name);  
  else {  
    console.log(n.name);  
    n.children.forEach(print);  
  }  
}
```

**Решение:** Лист и контейнер реализуют общий интерфейс.

**Код реализации:**

```ts
interface Component {  
  show(indent?: string): void;  
}  
  
class File implements Component {  
  constructor(private name: string) {}  
  
  show(indent = '') {  
    console.log(indent + this.name);  
  }  
}  
  
class Folder implements Component {  
  private children: Component[] = [];  
  
  constructor(private name: string) {}  
  
  add(c: Component) {  
    this.children.push(c);  
  }  
  
  show(indent = '') {  
    console.log(indent + this.name);  
    this.children.forEach((c) => c.show(indent + '  '));  
  }  
}  
  
const root = new Folder('root');//  /root
root.add(new File('a.txt'));    //  /root/a.txt
const docs = new Folder('docs');//  /roo  
docs.add(new File('readme.md'));  
root.add(docs);  
root.show();
```

---

## 10) Proxy

**Проблема:** Нужен контроль доступа/кеш/ленивая загрузка вокруг сервиса.  
**Симптом:** Кеш/проверки прав дублируются в каждом месте вызова.

**Плохой пример (симптом в коде):**

```ts
const data = service.getData();
if (!cache["data"]) cache["data"] = data; // дублируем в контроллерах, сервисах и т.д.
```

**Решение:** Ввести «заместителя», инкапсулирующего сквозную логику.

**Код реализации:**

```ts
interface Service {  
  getData(): string;  
}  
  
class RealService implements Service {  
  getData() {  
    return 'секрет';  
  }  
}  
  
class ServiceProxy implements Service {  
  private cache?: string;  
  
  constructor(private real = new RealService()) {}  
  
  getData() {  
    if (!this.cache) {  
      this.cache = this.real.getData();  
      console.log('real');  
    } else console.log('cache');  
    return this.cache;  
  }  
}  
  
const svc = new ServiceProxy();  
svc.getData(); // real  
svc.getData(); // cache
```

---

# Поведенческие

## 11) Strategy

**Проблема:** Нужно переключать алгоритмы (сортировка, ценообразование, доставка).  
**Симптом:** Большие `if/else` по типу алгоритма.

**Плохой пример (симптом в коде):**

```ts
function price(base: number, region: 'EU' | 'US' | 'RU') {  
  if (region === 'EU') return base * 1.2;  
  if (region === 'US') return base * 1.1;  
  return base * 1.05;  
}
```

**Решение:** Выделить стратегии, подставлять нужную во время исполнения.

**Код реализации:**

```ts
interface PricingStrategy {  
  calc(base: number): number;  
}  
  
enum PricingType {  
  EU,  
  US  
}  
  
class EUPricing implements PricingStrategy {  
  calc(b: number) {  
    return b * 1.2;  
  }  
}  
  
class USPricing implements PricingStrategy {  
  calc(b: number) {  
    return b * 1.1;  
  }  
}  
  
class Checkout {  
  constructor(private readonly strategyTable: Map<PricingType, PricingStrategy>) {}  
  
  total(type: PricingType, base: number) {  
    return this.strategyTable.get(type)!.calc(base);  
  }  
}  
  
const checkout = new Checkout(  
  new Map<PricingType, PricingStrategy>([  
    [PricingType.EU, new EUPricing()],  
    [PricingType.US, new USPricing()]  
  ])  
);  
  
checkout.total(PricingType.EU, 3000);
```

---

## 12) Observer

**Проблема:** Много подписчиков должны реагировать на изменение (события, курсы валют, инвентарь).  
**Симптом:** Ручные вызовы каждого зависимого компонента.

**Плохой пример (симптом в коде):**

```ts
function onPriceChange(v:number){
  updateUI(v);
  sendAnalytics(v);
  notifyPartners(v); // растёт список вызовов
}
```

**Решение:** Модель «подписка-уведомление».

**Код реализации:**

```ts
type Listener = (v: number) => void;  
  
class Subject {  
  private observers = new Set<Listener>();  
  
  subscribe(l: Listener) {  
    this.observers.add(l);  
  }  
  
  unsubscribe(l: Listener) {  
    this.observers.delete(l);  
  }  
  
  notify(v: number) {  
    this.observers.forEach((l) => l(v));  
  }  
}  
  
const price$ = new Subject();  
price$.subscribe((v) => console.log('UI:', v));  
price$.subscribe((v) => console.log('Analytics:', v));  
price$.notify(120);
```

---

## 13) Command

**Проблема:** Нужны undo/redo, очередь задач, лог аудита.  
**Симптом:** Действия вызываются напрямую без истории.

**Плохой пример (симптом в коде):**

```ts
class Cart { items:string[]=[]; add(i:string){ this.items.push(i);} }
const cart = new Cart();
cart.add("Телефон");
// Как откатить? Истории нет.
```

**Решение:** Инкапсулировать действия в команды и хранить историю.

**Код реализации:**

```ts
interface Command {  
  execute(): void;  
  
  undo(): void;  
}  
  
class Cart {  
  items: string[] = [];  
  
  add(i: string) {  
    this.items.push(i);  
  }  
  
  remove(i: string) {  
    this.items = this.items.filter((x) => x !== i);  
  }  
}  
  
class AddItemCommand implements Command {  
  constructor(  
    private cart: Cart,  
    private item: string
  ) {}  
  
  execute() {  
    this.cart.add(this.item);  
  }  
  
  undo() {  
    this.cart.remove(this.item);  
  }  
}  
  
class Invoker {  
  private history: Command[] = [];  
  
  run(c: Command) {  
    c.execute();  
    this.history.push(c);  
  }  
  
  undo() {  
    this.history.pop()?.undo();  
  }  
}  
  
const inv = new Invoker();  
const cart2 = new Cart();  
inv.run(new AddItemCommand(cart2, 'Телефон'));  
inv.undo();
```

---

## 14) State

**Проблема:** Поведение объекта зависит от состояния (статус заказа, дверь, документ).  
**Симптом:** Гигантский `switch(state)` в каждом методе.

**Плохой пример (симптом в коде):**

```ts
type DoorState = 'locked' | 'unlocked';  
let state: DoorState = 'locked';  
  
function click() {  
  if (state === 'locked') {  
    console.log('open');  
    state = 'unlocked';  
  } else {  
    console.log('close');  
    state = 'locked';  
  }  
}
```

**Решение:** Вынести состояния в отдельные классы, контекст делегирует обработку.

**Код реализации:**

```ts
interface State {  
  handle(ctx: Context): void;  
}  
  
class Context {  
  constructor(public state: State) {}  
  
  set(s: State) {  
    this.state = s;  
  }  
  
  request() {  
    this.state.handle(this);  
  }  
}  
  
class Locked implements State {  
  handle(ctx: Context) {  
    console.log('Открываем');  
    ctx.set(new Unlocked());  
  }  
}  
  
class Unlocked implements State {  
  handle(ctx: Context) {  
    console.log('Закрываем');  
    ctx.set(new Locked());  
  }  
}  
  
const door = new Context(new Locked());  
door.request();  
door.request();
```

---

## 15) Chain of Responsibility

**Проблема:** Запрос должен пройти через цепочку проверок (аутентификация, валидация, бизнес-правила).  
**Симптом:** Вложенные `if`, размазанные по методу.

**Плохой пример (симптом в коде):**

```ts
function handle(req:string){
  if (!req.includes("auth")) return "401";
  if (!req.includes("payload")) return "400";
  return "200";
}
```

**Решение:** Сборка цепочки обработчиков — каждый либо обрабатывает, либо передаёт дальше.

**Код реализации:**

```ts
abstract class Handler {  
  private next?: Handler;  
  
  setNext(h: Handler) {  
    this.next = h;  
    return h;  
  }  
  
  handle(req: string): string | undefined {  
    return this.next?.handle(req);  
  }  
}  
  
class AuthHandler extends Handler {  
  handle(req: string) {  
    if (!req.includes('auth')) return '401';  
    return super.handle(req);  
  }  
}  
  
class ValidationHandler extends Handler {  
  handle(req: string) {  
    if (!req.includes('payload')) return '400';  
    return super.handle(req) ?? '200';  
  }  
}  
  // {} -> {} -> {} -> {}
const pipeline = new AuthHandler();  
pipeline.setNext(new ValidationHandler());  
console.log(pipeline.handle('auth+payload')); // 200  
console.log(pipeline.handle('payload')); // 401
```