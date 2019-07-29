Table des matières :

1. [Introduction](#introduction)
2. [Qu’est ce qu’un hook ?](#qu-est-ce-qu-un-hook)
3. [Pourquoi utiliser les hooks ?](#pourquoi-utiliser-les-hooks)
4. [API](#api)
   - [`useState()`](#use-state)
   - [`useEffect()`](#use-effect)
   - [`useLayoutEffect()`](#use-layout-effect)
   - [`useRef()`](#use-ref)
   - [`useMemo()`](#use-memo)
   - [`useReducer()`](#use-reducer)
   - [`useContext()`](#use-context)
5. [Les customs hooks](#les-customs-hooks)
6. [Limitations et règles d'utilisations](#limitations)
7. [Liens utiles](#liens-utiles)

## Introduction <a name="introduction"></a>

Les hooks React, disponible depuis la version 16.8 permette le management de states et d’effets (sorte de lifecycle) au sein d’un composant fonctionnel. Cela n’était précédemment possible qu’au sein des class component. Ces derniers ont été implémentés au sein de la mise à jour de React nommée “React Fiber” en février 2019.

L’implémentation de cette nouvelle API n’engage aucun “breaking change”. L’utilisation des composant développés en class components sont toujours possible, bien que l’on ne puisse utiliser les hooks dans une class.

[Documentation : Introduction aux hooks](https://reactjs.org/docs/hooks-intro.html)

## Qu’est ce qu’un hook ? <a name="qu-est-ce-qu-un-hook"></a>

Un hook React est une fonction rendu disponible par l’API React permettant de manager des states et des effets au sein d’une fonction. Un hook est toujours préfixé par le mot *use*. Exemple : useMaFunction.

## Pourquoi utiliser les hooks ? <a name="pourquoi-utiliser-les-hooks"></a>

Cette nouvelle API va nous permettre de splitter intelligemment différentes logiques de notre composant, voir dans certain cas, de les externaliser et de les réutiliser à travers n'importe quel composant de notre application. (ces fonctions/logiques externalisées sont nommées *custom Hooks*). 

L'apport de cette feature réduit concidérablement le poid final du bundle compilé mais aussi les répétitions de code (exemple : vérifier que nous sommes sur un device mobile ou non et écouter le resize du device).

## API <a name="api"></a>

Voici les hooks que nous allons utiliser le plus souvent :

### `useState()`  <a name="use-state"></a>

Cette fonction permet de seter un state “réactif” au sein d’une fonction : 

```js
const [counter, setCounter] = useState(0);
```

`useState()` est une fonction retournant un tableau :

- la première entrée `counter` est le nom du state.
- la seconde entrée `setCounter` est le seter que nous appellerons pour modifier ce state. Celle-ci sera toujours préfixée du mot *set* suivi du nom de la function. 
- `0`, est la valeur initiale du state

Pour muter un state, nous utilisera le seter : 

```js
setCounter(nouvelleValue);
```

En typescript, nous typerons chaque state de cette manière :

```js
const [counter, setCounter] = useState<number>(0);
```

> Note : le state utilisant le hook useState reste asynchrone, tout comme dans les "class components".

[Documentation useState](https://reactjs.org/docs/hooks-state.html)

### `useEffect()` <a name="use-effect"></a>

`useEffect` permettent de gérer l’ensemble des “effets de bord” du hook. En d’autres termes, tout ce que que le composant va déclanger une fois monté, updaté et démonté.

On retouvera *partiellement* au sein de cette fonction les "life cycles" disponible dans les class components.

#### 1er argument : ()=>

Le premier argument de la function `useEffect()` est une fonction qui exécutera du code **une fois le virtual DOM du composant rendu par React**. Cette même fonction retourne le code exécuté à la destruction du composant.

```js 
useEffect(()=> {
    // on mount 
    return () => {
        // on unmount
    }
});
```

Contrairement à `componentDidMount()`, le code passé à cet argument sera éxécuté à chaque fois que le DOM sera rendu par le composant. Nous aurons pourtant souvant besoin que ce code ne s'execute qu'une seule fois. Voici le seconde argument de `useEffect()`.

#### 2eme argument : []

`UseEffect()` reçoit un second argument qui est un tableau [].

```js 
useEffect(()=> {
    // ... Ce code ne sera exécuté :
    // - uniquement au premier montage du composant (pas à son update).
}, []);
```

 Ce tableau permet à React de faire la diff entre les deux états du composant entre chaque rendu. Si l'on spécifie un tableau vide, React ne voit pas de différence entre les deux états et ne re-exécute pas le code de la fonction passé en premier argument.

Dans le cas où nous aurions besoin que ce code ne s'exécute à chaque fois que le composant voit l'un de ses states ou props muté : 

```js 
useEffect(()=> {
    // ... Ce code ne sera exécuté :
    // - au premier "mount" du composant 
    // - et lorsque le state `counter` aura muté
}, [counter]);
```

La seule manière d'éxecuter le contenu du premier argument de `useEffect()`**uniquement à son update** est d'utiliser une référence qui fera office de "flag".

```js 
// créer une ref 
const initialMount = useRef(true);
// créer l'effet 
useEffect(()=> {
    if ( initialMount.current ) {
        // c'est la première exécution de cet effet.
        // modifier la valeur de la ref servant de flag
        initialMount.current = false;
    } else {
        // ...le code ici sera a chaque fois que le state counter changera, sauf la première fois que le composant passera sur cet effet 
    }
}, [counter]);
```

***Important*** : 

- Il ne faut pas oublié que React rend de manière "transparente" le DOM à chaque fois que le composant voit l'un de ses states ou props modifié ! Ce qui veut dire que `UseEffect()`, sans deuxième argument, sera exécuté à chaque fois qu'un state ou une props change. Pour plus d'information sur le sujet, lire l'article de Dan Abramov [A complete Guide to useEffect](https://overreacted.io/a-complete-guide-to-useeffect/). 
- `useEffect()` ne bloque pas le thread. Il sera donc exécuté seulement une fois que le DOM sera rendu.

[Documentation : useEffect()](https://reactjs.org/docs/hooks-effect.html)

### `useLayoutEffect()` <a name="use-layout-effect"></a>

Ce hook est une variante de `useEffect()`. La seule différence est que celui-ci bloquera le rendu du thread, le DOM sera rendu après l'éxécution de la function passé en premier argument. Cette variante s'avère pratique lorsqu'il s'agit d'animer l'entré d'un composant.

[Documentation : useLayoutEffect()](https://reactjs.org/docs/hooks-reference.html#uselayouteffect)

### `useRef()` <a name="use-ref"></a>

Ce hook permet de créer une référence DOM.

```js
function Bar() {
    // créer un référence nommée "root", sa valeur est "null" à son initialisation
    const root = useRef(null);

    // exécuté une fois le DOM rendu
    useEffect(()=> {
        // le noeuf DOM se trouve dans la clef "current" de "root"
        console.log( root.current ); // -> <div class="Bar" /> 
    })

    // retourner du DOM 
    return <div className="Bar" ref={root} />
}
```

Dans le cas où l'on souhaiterait récupérer un tableau de ref, si par exemple la ref se trouve dans dans un `map` : [get d'un tableau de référence DOM](https://codesandbox.io/s/get-array-of-ref-2-ocddc)

[Documentation : useRef()](https://reactjs.org/docs/hooks-reference.html#useref)

### `useMemo()` <a name="use-memo"></a>

UseMemo va nous permettre de "mémoïser" un appel d'une fonction ou le contenu d'une variable. Cela est particulièrement utile si l'on prend en compte l'environnement fonctionnel. Pour rappel, à chaque mutation de state ou props, l'ensemble de la fonction va être re-exécuté. 

Example d'une instance, qui, écrit de la sorte au sein d'une fonction, va être ré-appelé entre chaque rendu :
```js
const instance = new ClassInstance();
```

Pour éviter cela, nous pouvons mettre en mémoire cette variable contenant l'instance, pour que celle-ci ne soit pas appelé à nouveau entre chaque rendu, ou du moins, ne soit rendu qu'en fonction du changement d'une valeur précise (pour cela, renseigner une valeur dans le tableau en second paramètre)

```js
const instance = useMemo(()=> new ClassInstance(), []);
```

[Documentation : useMemo()](https://reactjs.org/docs/hooks-reference.html#usememo)

### `useReducer()` <a name="use-reducer"></a>

Utile lorsque l'on a besoin de manager plusieurs states qui (fonctionnent, intéragissent) ensemble, ou quand le state suivant dépend du précédent.

> fonctionne comme la fonction `reduce()` native javascript.

```js
// définir une fonction de transformation pour le state
// qui prend deux arguments state et action 
function fonctionDeTransformation (state, action) {
    // si l'action dispatchée est le string "coucou"
    if (action === "coucou") {
        // muter le state
        return state + 3;
    }
}

// définir un state initial 
const initialState = 0;
// init du state count
const [count, dispatch] = useReducer(fonctionDeTransformation, initialState); 
```

```js
// utiliser le dispatcher pour muter le state
<div onClick={()=> dispatch('coucou')}> // le click retournera state + 3
```

- [Documentation : useReducer()](https://reactjs.org/docs/hooks-reference.html#usereducer)
- [Test codeSandBox](https://codesandbox.io/s/test-usereducer-ycmvz)
- [article "how to use useReducer in react hooks"](https://medium.com/crowdbotics/how-to-use-usereducer-in-react-hooks-for-performance-optimization-ecafca9e7bf5)
- [tutoriel créer un panier shop avec useReducer](https://daveceddia.com/usereducer-hook-exemples/)

### `useContext()`  <a name="use-context"></a>

Permet de storer un context global accessible depuis n'importe quel composant de l'application. 

[Documentation : useContext()](https://reactjs.org/docs/hooks-reference.html#usecontext)

### Autres hooks <a name="autres-hooks"></a>

Il exite plusieurs autre fonctions disponibles dans l'API. Dont certaines que l'on utilisera moins souvent. 

[Documentation de l'API hooks de React](https://reactjs.org/docs/hooks-reference.html)

## Les customs hooks <a name="les-customs-hooks"></a>

Les custom hooks sont des functions qui contiennent une logique utilisable depuis n'importe quel composant. Cette possibilité va nous mener à créer notre propre collection de *custom hooks* au titre de helpers.

Il existe déjà plusieurs collections de *custom hooks* open source : 

- [react-hooks collection](https://nikgraf.github.io/react-hooks/)
- [rooks](https://github.com/imbhargav5/rooks)
- [useHooks](https://usehooks.com)

## Limitations et règles d'utilisations  <a name="limitations"></a>

- Un hook ne peut être appelé qu'à l'intérieur d'un composant fonctionnel (pas dans une class component).
- N'importe quel hook doit être appelé au top level du composant. Il ne doit jamais être imbriqué dans une condition ou appelé dans un autre hook.

[Documentation : Rules of Hooks](https://reactjs.org/docs/hooks-rules.html)

## Liens utiles <a name="liens-utiles"></a>

- [awesome-react-hooks](https://github.com/rehooks/awesome-react-hooks)



