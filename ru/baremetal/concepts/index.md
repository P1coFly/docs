# Обзор сервиса {{ baremetal-full-name }}

{{ baremetal-full-name }} предоставляет возможность полностью арендовать физический сервер, выбрав его среди готовых, доступных к заказу, конфигураций.

Сервис позволяет удаленно установить операционную систему на сервер из заранее подготовленных образов c {{ marketplace-short-name }} или загрузить в {{ objstorage-full-name }} собственный образ и использовать его для установки. Доступ к серверу может осуществляться с помощью KVM-консоли или через SSH.

Все серверы имеют подключение к публичной сети интернет и к приватной сети. В приватной сети можно создавать приватные подсети и VRF, в которых можно объединять серверы, предназначенные для выполнения определенных типов задач.

## Серверы и сети {#concepts}

* [{#T}](./servers.md)
* [{#T}](./server-configurations.md)
* [{#T}](./network.md)
