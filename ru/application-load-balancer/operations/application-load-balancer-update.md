# Изменить L7-балансировщик

Чтобы изменить L7-балансировщик:

{% list tabs %}

- Консоль управления

  1. В [консоли управления]({{ link-console-main }}) выберите каталог, которому принадлежит балансировщик.
  1. Выберите сервис **{{ alb-name }}**.
  1. Нажмите на имя нужного балансировщика.
  1. Нажмите **Редактировать**.
  1. Измените параметры балансировщика, например, переименуйте балансировщик.
  1. Внизу страницы нажмите кнопку **Сохранить изменения**.

- CLI

  {% include [cli-install](../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../_includes/default-catalogue.md) %}

  1. Посмотрите описание команды CLI для изменения балансировщиков:

     ```
     yc alb load-balancer update --help
     ```

  1. Выполните команду, указав новые параметры балансировщика:

     ```
     yc alb load-balancer update <имя балансировщика> --new-name <новое имя балансировщика>
     ```

{% endlist %}