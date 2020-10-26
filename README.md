
# Cycle PushState Driver

A [Cycle.js](http://cycle.js.org) [driver](http://cycle.js.org/drivers.html) for the [History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API).

## Opções de Design

Este é um driver Cycle.js mínimo que simplesmente isola chamadas `` `history.pushState``` e eventos` `` popstate```. Não faz nenhuma suposição sobre como você deseja filtrar e transformar fluxos de eventos de clique / toque para criar os caminhos a serem enviados. Nem fará `` ʻe.preventDefault () `` `para você.

Na ausência de suporte ``pushState``, ele permite que os links funcionem como links normais. Isso significa que o driver ``preventDefault`` precisa saber para não capturar esses eventos de clique / toque.

Finalmente, ele não oferece suporte atualmente a ``history.replaceState`` ou o argumento de estado de ``pushState``

If you prefer a driver that covers all these cases, you may want to consider the [```TylorS/cycle-history```](https://github.com/TylorS/cycle-history) driver which is a batteries-included approach to the same problem.

Se você preferir um driver que cubra todos esses casos, pode considerar o [```driver TylorS / histórico do ciclo```](https://github.com/TylorS/cycle-history) que é uma abordagem incluída com baterias para o mesmo problema.

## API

### ```makePushStateDriver ()```

Retorna um driver que chama ``history.pushState`` nos caminhos de entrada e caminhos de saída enviados para ``pushState``, bem como recebidos com eventos ``popstate``, começando com o caminho atual. Se ``pushState`` não for suportado, esta função retorna um driver que simplesmente emite o caminho atual.

## Instalar

```sh
npm install cycle-pushstate-driver
```

## Uso

Basics:

```js
import Cycle from '@cycle/core'
import { makePushStateDriver } from 'cycle-pushstate-driver'

function main (responses) {
  // ...
}

const drivers = {
  Path: makePushStateDriver()
}

Cycle.run(main, drivers)
```

Caso de uso simples:

```js
function main({ DOM, Path }) {
  let localLinkClick$ = DOM.select('a').events('click')
    .filter(e => e.currentTarget.host === location.host)

  let navigate$ = localLinkClick$
    .map(e => e.currentTarget.href)

  let vtree$ = Path
    .map(url => {
      switch(url) {
        case '/':
          renderHome()
          break
        case '/user':
          renderUser()
          break
        default:
          render404()
          break
      }
    })

  return {
    DOM: vtree$,
    Path: navigate$,
    preventDefault: localLinkClick$
  };
}
```

Roteamento de caso de uso com ```switch-path```:

```js
import switchPath from 'switch-path'
import routes from './routes'

function resolve (path) {
  return switchPath(path, routes)
}

function main({ DOM, Path }) {
  const localLinkClick$ = DOM.select('a').events('click')
    .filter(e => e.currentTarget.host === location.host)

  const navigate$ = localLinkClick$
    .map(e => e.currentTarget.href)

  const vtree$ = Path
    .map(resolve)
    .map(({ value }) => value)

  return {
    DOM: vtree$,
    Path: navigate$,
    preventDefault: localLinkClick$
  };
}
```

Roteamento de caso de uso com ```wayfarer```:
```js
import wayfarer from 'wayfarer'

function route (path$) {
  const route$ = new Rx.ReplaySubject(1)
  const r = name => params => route$.onNext({ name, params })

  const router = wayfarer('/notfound')
  router.on('/', r('owers'))
  router.on('/owers/:ower', r('owees'))
  router.on('/notfound', r('notfound'))

  path$
    .subscribe(
      path => router(path),
      route$.onError.bind(route$),
      route$.onCompleted.bind(route$)
    )
  return route$
}

function main({ DOM, Path }) {
  const Route = route(Path)
  
  const localLinkClick$ = DOM.select('a').events('click')
    .filter(e => e.currentTarget.host === location.host)

  const navigate$ = localLinkClick$
    .map(e => e.currentTarget.href)

  const vtree$ = Rx.Observable.combineLatest(
    Route, owersVtree$, oweesVtree$, notfoundVtree$,
    (route, owersVtree, oweesVtree, notfoundVtree) => {
      const vtrees = {
        'owers': owersVtree,
        'owees': oweesVtree,
        'notfound': notfoundVtree
      }
      return vtrees[route.name]
    }
  )

  return {
    DOM: vtree$,
    Path: navigate$,
    preventDefault: localLinkClick$
  };
}
```
