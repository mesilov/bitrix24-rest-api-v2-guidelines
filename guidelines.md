# Гайдлайны к REST-API для Битрикс24 v2

## 1. Назначениие документа
Содержит описание принципов работы REST-API v2 для Битрикс24. Принципиальные решения предлагаются партнёрами и обсуждаются на рабочих встречах совместно с представителями вендора.

## 2. Оглавление
1. Безопасность
2. Формат данных
3. Правила построения URL
4. HTTP-запросы
5. HTTP-статус коды
6. HTTP-заголовки
7. Кеширование и производительность
8. Обратная совместимость
9. Вывод из эксплуатации - deprecations
10. События

## 3. Исходные наработки от вендора
1. Методы должны иметь последним элементом (после последней точки) слово, удовлетворяющее нашим правилам именования методов
```
disk.file.get - ХОРОШО 
disk.file.informationAboutFile - ПЛОХО
```

2. Стандартные методы CRUD (если они есть) имеют стандартные имена  `add`, `update`, `delete`, `list`

Параметры стандартных методов стандартизованы
```
module.entity.get(id)
module.entity.update(id, fields)
module.entity.delete(id)
module.entity.list([select, group], filter, order)
//примеры
disk.folder.get(id)
sale.order.update(id, fields)

```
3. Если метод оперирует некой главной сущностью, то её параметр-идентификатор именуется, как `id`

```javascript
BX24.callMethod("disk.folder.get", {id: 12 });
```
```php
public function getAction(Folder $folder);
public function getAction($id); //если не нужен autowiring
```
```javascript
BX24.callMethod("sale.order.update", {id: 560 });
```

```php
public function updateAction(Order $order);
public function updateAction($id); //если не нужен autowiring
```

4. Если метод оперирует вспомогательной сущностью, то её параметр-идентификатор именуется, как `{entityName}Id`

```javascript
BX24.callMethod("disk.folder.attachToStorage", {
          id: 12,
         storageId: 3
});
```
```php
public function attachToStorageAction(Folder $folder, Storage $storage);
```

5. Используется только верблюжья нотация (camelCase), соответствующая общей стилистике продукта. Как в ответах, так и во входных данных

```javascript
BX24.callMethod("module.entity.list", { filter: {
name: "Etalon",
          parentId: 100
      }
 })
```
Пример ответа:
```
order: {
          id: 1,
          name: "My Toys",
          basket: {
                    items: [ 
                              {
                                        id: 1,
                                        productId: 100,
                              },
                              {
                                        id: 2,
                                        productId: 2140,
                              }
                           ]
                              }
                  },
  debugInfo: 
          {
          }
```

6. В запросах и ответах считаем, что пользователь не знает, какие результаты мы берем из базы, а какие вычисляем и формируем структуру в зависимости от физического смысла значений, а не от способа их хранения или получения. 


ХОРОШО
```
user: {
          id: 5,
          name: "Вася",
          calculatedSum: 28,
          links: {
                    detail: "https://...",
                    list: "https://..." 
                 }
}
```
Плохо:
```
order: 
          {
                    id: 1,
                    name: "My Toys",
                    buyer:
                              {
                                        id: 42,
                                        name: "Вася",
                                        lastName: null
                              },
                    payments: [],
                    basket: 
                              {
                                        items: [ 
                                                  {
                                                            id: 1,
                                                            productId: 100,
                                                  },
                                                  {
                                                            id: 2,
                                                            productId: 2140,
                                                  }
                                               ] }
                              },
debugInfo: {
}
```

7. Всегда в ответе на первом уровне ключем идёт сущность или коллекция, к которой идёт обращение
```
//user.get

user: { 
    id: 5, 
    name: "Вася", 
    links: { 
        detail: "https://...", 
        list: "https://..." 
    }
} 

//user.list 

users: [ 
    {
        id: 5, 
        name: "Вася", 
        links: { 
            detail: "https://...", 
            list: "https://..." 
        } 
    }, 
    {
        id: 51,
        name: "Aся",
        links: { 
            detail: "https://...", 
            list: "https://..." 
        } 
    } 
]
```

8. Связные сущности идут вложенными объектами. Если это коллекция, то это массив и поле именуется в множественном числе. Если коллекция пустая, то возвращаем пустой массив.
```
order: { 
    id: 1, 
    name: "My Toys",
    buyer: { 
        id: 42,
        name: "Вася", 
        lastName: null 
    }, 
    payments: [], 
    basket: { 
        items: [ 
            { 
                id: 1, 
                productId: 100, 
            }, 
            { 
                id: 2, 
                productId: 2140, 
            } 
        ] 
    }
}, 
debugInfo: {
} 
```

9. Если в ответе есть ссылки, то предусматриваем именование, используя слова `link`, `links`. Рекомендуем даже в случае одной ссылки делать коллекцию `links` будущее.
```
file: 
          {
                    id: 5,
                    name: "Вася.docx", 
                    links: {
                              download: "https://...",
                              detail: "https://..." 
                           }
          }
```

10. Дата и время должны быть в формате ATOM (ISO-8601).
11. Рекомедуем преобразовывать строковые данные из   к явным типам. То есть число - сделать числом, булевый тип - булевым, null - null'ом.

