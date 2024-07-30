# Component Store

## Предисловие

Мы используем `Component Store`-паттерн как частную локальную альтернативу полноценным `ngrx` сторам.
Если ваш раздел не велик, состоит из одной странички и полноценный стор кажется избыточным - не стоит идти в "angular way"
архитектуру, так как "переключение" между столь разными паттернами может вызывать трудности в восприятии у других разработчиков.
Вместо этого используйте `Component Store` - тогда весь наш проект будет написан в "одном ключе".

## Документация
[Вот тут](https://ngrx.io/guide/component-store). Наибольший интерес представляют разделы "Architecture" и "Usage".

## Организация стора

### Структура модуля с Component Store

Реализация паттерна должна состоять из двух файлов:

* foo.store.ts
* foo.facade.ts

Все методы и поля реализующие взаимодействие со стором публичны, но напрямую используются только в фасаде.
Обращение к "внутренностям" стора откуда-либо кроме фасада запрещено. Таким образом фасад берёт на себя
роль "инкапсулятора" стора.

Полностью дерево раздела может выглядеть следующим образом

```
/modules
  /foo
    /pages
      /foo-page
        foo-page.component.ts
        ...
    /services
      foo.store.ts
      foo.facade.ts
      ...
    foo.module.ts
    ...
```

### Подключение

Вам достаточно "запровайдить" сервисы стора и фасада в page компоненте и заинжектить фасад туда же (см. пример кода ниже)

## Оформление стора

Порядок записи элементов стора в реализующем его классе следующий:

* Updaters
* Selectors
* Effects
* constructor with injects
* private methods (никакого взаимодействия непосредственно со стором, только вспомогательные методы)

Так же старайтесь располагать идентичные элементы в порядке нарастания сложности.
Чем проще updater, selector, effect - тем он выше в соответствующем разделе стора.

### Пример

#### Store

```ts
// foo.store.ts
import {Injectable} from '@angular/core';
import {ComponentStore} from '@ngrx/component-store';
import {FooMeta} from '../interfaces/foo-meta';
import {Bar} from '...';
import {BarData} from '...';
import {FooService} from './foo.service';

interface FooState {
    bar: Bar;
    baz: string;
    meta: FooMeta;
}

const initialState: FooState = {
    bar: null,
    baz: null,
    meta: {
        isFetching: false,
        isSubmitting: false,
    }
};

@Injectable()
export class FooStore extends ComponentStore<StoreSharedOrderState> {
    // -------------------------------------------------------------------------------------------------------- UPDATERS
    readonly setBar = this.updater((state, bar) => ({...state, bar}));

    readonly setBarField = this.updater((state, barField) => ({
        ...state,
        bar: {...state.bar, barField},
    }));

    readonly setIsFetching = this.updater((state, isFetching) => ({
        ...state,
        meta: {...state.meta, isFetching},
    }));

    readonly setIsSubmitting = this.updater((state, isSubmitting) => ({
        ...state,
        meta: {...state.meta, isSubmitting},
    }));

    // other updaters here

    // ------------------------------------------------------------------------------------------------------- SELECTORS
    readonly bar$: Observable<string> = this.select((state) => state.bar);

    readonly meta$: Observable<FooMeta> = this.select((state) => state.meta);

    // --------------------------------------------------------------------------------------------------------- EFFECTS
    readonly fetchBar = this.effect<void>((trigger$) => {
        return trigger$.pipe(
            tap(() => this.setIsFetching(true)),
            switchMap(() => this.fooService.fetchBar().pipe(
                tap((baz) => {
                    this.setIsFetching(false);
                    this.setBar(baz);
                }),
                catchError((error: HttpErrorResponse) => {
                    this.setIsFetching(false);
                    return this.handleError(error);
                }),
            )),
        );
    });

    readonly submitBar = this.effect((trigger$: Observable<Bar>) => {
        return trigger$.pipe(
            tap(() => this.setIsSubmitting(true)),
            map((bar) => this.makeBarData(bar)),
            switchMap((data) => this.fooService.submitBarData(data).pipe(
                tap(() => {
                    this.setIsSubmitting(false);
                }),
                catchError((error: HttpErrorResponse) => {
                    this.setIsSubmitting(false);
                    return this.handleError(error);
                }),
            ))
        );
    });

    // ----------------------------------------------------------------------------------------------------- CONSTRUCTOR
    constructor(
        private readonly fooService: FooService,
    ) {
        super(initialState);
    }

    // --------------------------------------------------------------------------------------------------------- PRIVATE
    private makeBarData(bar: Bar): BarData {
       // some private manipulations
       const data: BarData = {
           someField: bar.barField,
       };

       return data;
    }

    private handleError(error: HttpErrorResponse): void {
       // some error handling
    }
}
```

#### Facade

```ts
// foo.facade.ts
import {Injectable} from '@angular/core';
import {FooStore} from './foo.store';
import {FooMeta} from '../interfaces/foo-meta';
import {Bar} from '...';

@Injectable()
export class FooFacade {
    readonly bar$: Observable<string> = this.fooStore.bar$;
    readonly meta$: Observable<FooMeta> = this.fooStore.meta$;

    constructor(
        private readonly fooStore: FooStore,
    ) {}

    fetchBaz(): void {
        this.fooStore.fetchBaz();
    }

    submitBar(bar: Bar): void {
        this.fooStore.submitBar(bar);
    }
}
```

#### Component with Component Store

```ts
// foo-page.component.ts

@Component({
  ...
  providers: [FooStore, FooFacade],
})
export class FooPageComponent implements OnInit {
    readonly bar$: Observable<Bar> = this.fooFacade.bar$;
    readonly meta$: Observable<FooMeta> = this.fooFacade.meta$;

    constructor(
        private readonly fooFacade: FooFacade
    ) {}

    ngOnInit(): void {
      this.fooFacade.fetchBar();
    }

    onSomething(bar: Bar): void {
        this.fooFacade.submitBar(bar);
    }
}
```

## P.S.

### Плюсы
При соблюдении данных правил мы получаем легко читаемый код, с визуально отделёнными друг от друга составляющими.
Реализация фасадов и взаимодействие компонентов со стором не отличаются от классических `ngrx`-сторов, а значит
при необходимости можно будет без переделывания всего и вся подменить одну реализацию стора на другую.

### Нюансы
Помимо `updater` метода у класса `ComponentStore` есть методы `setState` и `patchState` - их использование у нас
запрещено, так как нарушается консистентность и страдает читабельность.
