# Гайдлайны к REST-API для Битрикс24 v2

## 1. Назначениие документа
Содержит описание принципов работы REST-API v2 для Битрикс24. Принципиальные решения предлагаются партнёрами и обсуждаются на рабочих встречах совместно с представителями вендора.


## 2. Референсы на подобные документы
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md)

## Исходные наработки от вендора
Методы должны иметь последним элементом (после последней точки) слово, удовлетворяющее нашим правилам именования методов
```
disk.file.get - ХОРОШО 
disk.file.informationAboutFile - ПЛОХО
```

Стандартные методы CRUD (если они есть) имеют стандартные имена  `add`, `update`, `delete`, `list`

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
Если метод оперирует некой главной сущностью, то её параметр-идентификатор именуется, как `id`
