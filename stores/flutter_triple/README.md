# flutter_triple

Implementação do Segmented State Pattern (Padrão de Estado Segmentado) apelidado de Triple.


## Segmentação do Estado

O SSP segmenta o estado em 3 partes reativas, o valor do estado (state), o objeto de erro (error) e a ação de carregamento do estado (loading).

.

![Triple](https://github.com/Flutterando/triple_pattern/raw/master/schema.png)

Esses segmentos são observados em um listener ou em listeners separados. Também podem ser combinados para se obter um novo segmento, sempre partindo dos 3 segmentos principais.

## Sobre o Package

Este package tem por objetivo introduzir Stores no padrão de segmentos pré-implementadas usando a API de Streams(StreamStore) e do objeto ValueNotifier (NotifierStore).

As Stores já oferecem por padrão um observador (**store.observer()**) e os métodos **store.undo()** e **store.repo()** que utilizam o design pattern **Memento** para desfazer ou refazer o valor do estado.

O Package também conta com **Builder Widgets** para observar as modificações do estado na árvore de widget do Flutter.

## Gerênciando o Estado com Streams

Para criar uma Store que ficará responsável pela Lógica do estado, crie uma classe e herde de **StreamStore**:

```dart
class Counter extends StreamStore {}
```

Você também pode colocar tipos no valor do estado e no objeto de exception que iremos trabalhar nesse Store:

```dart
class Counter<int, Exception> extends StreamStore<int, Exception> {}
```

Finalizamos atribuindo um valor inicial para o estado desse Store invocando o construtor da classe pai (super):

```dart
class Counter<int, Exception> extends StreamStore<int, Exception> {

    Counter() : super(0);
}
```

Temos disponível na Store 3 métodos para mudar os segmentos **(setState, setError e setLoading)**. Vamos começar incrementando o estado:

```dart
class Counter<int, Exception> extends StreamStore<int, Exception> {

    Counter() : super(0);

    void increment(){
        setState(state + 1);
    }
}
```

Esse código já é o suficiente para fazer o contador funcionar.
Vamos adicionar um pouco de código assincrono para apresentar os métodos **setError** e **setLoading**

```dart
class Counter<int, Exception> extends StreamStore<int, Exception> {

    Counter() : super(0);

    Future<void> increment() async {
        setLoading(true);

        await Future.delayer(Duration(seconds: 1));

        int value = state + 1;
        if(value < 5) {
            setState(value);
        } else {
            setError(Exception('Error: state not can be > 4'))
        }
        setLoading(false);
    }
}
```

Aqui experimentamos a mudança de estados e os outros segmentos de loading e error. 
> **NOTE**: Tudo o que foi mostrado aqui para o **StreamStore** também server para o **NotifierStore**.

Os 3 segmentos operam separados mas podem ser "escutados" juntos. Agora iremos ver como observar esse store.

## Observers and Builders

### observer

Podemos observar os segmentos de forma individual ou de todos usando o método **store.observer()**;

```dart
counter.observer(
    onState: (state) => print(state),
    onError: (error) => print(error),
    onLoading: (loading) => print(loading),
);
```
Já nos Widgets podemos escolher escutar em um Builder com Escopo ou escutar todas as modificações do Triple.

### ScopedBuilder

Use o **ScopedBuilder** para escutar os segmentos de forma individual ou de todos, semelhante ao que faz o método **store.observer()**;

```dart
ScopedBuilder(
    store: counter,
    onState: (context, state) => Text('$state'),
    onError: (context, error) => Text(error.toString()),
    onLoading: (context, loading) => CircularProgressIndicator(),
);
```

> **NOTE**: No ScopedBuilder O **onLoading** só é chamado quando for "true". Isso significa que se o estado for modificado ou for adicionado um erro, o widget a ser construido será o do **onState** ou do **onError**. Porém é muito importante modificar o Loading para "false" quando a ação de carregamento for completada. Os **observers** do Triple *NÃO PROPAGAM OBJETOS REPETIDOS* (mais sobre isso na sessão sobre **distinct**). Esse é um comportamento exclusivo do ScopedBuilder.

### TripleBuilder

Use o **TripleBuilder** para escutar todas as modificações dos segmentos e refleti-las na arvore de Widgets.

```dart
TripleBuilder(
    store: counter,
    builder: (context, triple) => Text('${triple.state}'),
);
```

> **NOTE**: O Builder do **TripleBuilder** é chamado quando há qualquer alteração nos segmentos. Seu uso é recomendado apenas se tiver interesse em escutar todos os segmentos ao mesmo tempo.

### Distinct

Por padrão, o observer da Store não reage a objetos repetidos. Esse comportamento é benéfico pois evita reconstruções de estado e notificações se o segmento não foi alterado.

É uma boa prática sobreescrever o **operation==** do valor do estado e error. Uma boa dica também é usar o package [equateble](https://pub.dev/packages/equatable) para simplificar esse tipo de comparação.

## Selectors

Podemos recuperar a reatividade dos segmentos de forma individual para transformações ou combinações. Temos então 3 selectors que podem ser recuperados como propriedades do Store: **store.selectState**, **store.selectError** e **store.selectLoading**.

O Tipo dos selectors muda dependendo da ferramenta reativa que estiver utilizando nos Stores. Por exemplo, se estiver usando **StreamStore** então seus selectors serão Streams, porém se estiver usando **NotifierStore** então seus selectors serão ValueListenable;

```dart
//StreamStore
Stream<int> myState$ = counter.selectState;
Stream<Exception> myError$ = counter.selectError;
Stream<bool> myLoading$ = counter.selectLoading;

//NotifierStore
ValueListenable<int> myState$ = counter.selectState;
ValueListenable<Exception> myError$ = counter.selectError;
ValueListenable<bool> myLoading$ = counter.selectLoading;

```

## Dúvidas e Problemas

O Canal de **issues** está aberto para dúvidas, reportar problemas e sugestões, não exite em usar esse canal de comunicação.

> **VAMOS SER REFERRENCIAS JUNTOS**







