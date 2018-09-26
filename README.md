# rec-reader

## Определения

## Мотивация

### Возможность подменять/мокать методы сервисов

В данный момент у нас используются сервисы вида

``` haskell
data Service = Service
  { serviceMethod :: Arg -> ConcreteMonad Result
  }
```

И с этим не было бы проблем, если бы в `ContreteMonad` не требовались
различные методы и сервисы из монады хендлера, которые не нужны в
тестах или бенчмарках. Хочется иметь возможность конструировать
`Service` для прода честный, а для тестов фейковый, не требующий во
внутренних монадах различных сервисов.

Иначе говоря, хочется подменять `ConcreteMonad` на произвольную
монаду. Однако, наивным способом сделать это не удается

``` haskell
data Service m = Service
  { serviceMethod :: Arg -> m Result
  }

type Handler m = ReaderT (Service (Handler m)) m
```

в результате чего получаем

``` text
    Cycle in type synonym declarations:
```

По этому `type` надо заменить на `newtype`

``` haskell
newtype RecReaderT s m a = RecReaderT (ReaderT (s (RecReaderT s m)) m a)
```
