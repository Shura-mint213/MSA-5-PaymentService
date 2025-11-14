# Таблица переходов состояний

| Исходное состояние       | Переходное состояние     | Событие                  |
|--------------------------|--------------------------|--------------------------|
| INITIATED                | CHARGED                  | INITIATE_SUCCESS         |
| INITIATED                | FAILED                   | INITIATE_FAILED          |
| CHARGED                  | INTERNAL_FRAUD_CHECK     | CHARGE_SUCCESS           |
| CHARGED                  | CHARGED                  | CHARGE_RETRY             |
| INTERNAL_FRAUD_CHECK     | EXTERNAL_FRAUD_CHECK     | INTERNAL_APPROVED        |
| INTERNAL_FRAUD_CHECK     | REFUNDING                | INTERNAL_REJECTED        |
| INTERNAL_FRAUD_CHECK     | FAILED                   | INTERNAL_ERROR           |
| EXTERNAL_FRAUD_CHECK     | FRAUD_MANUAL             | EXTERNAL_MANUAL          |
| EXTERNAL_FRAUD_CHECK     | CREDITING                | EXTERNAL_APPROVED        |
| EXTERNAL_FRAUD_CHECK     | REFUNDING                | EXTERNAL_REJECTED        |
| EXTERNAL_FRAUD_CHECK     | EXTERNAL_FRAUD_CHECK     | EXTERNAL_RETRY           |
| FRAUD_MANUAL             | CREDITING                | MANUAL_APPROVED          |
| FRAUD_MANUAL             | REFUNDING                | MANUAL_REJECTED          |
| CREDITING                | COMPLETED                | CREDIT_SUCCESS           |
| CREDITING                | REFUNDING                | CREDIT_FAILED            |
| REFUNDING                | REFUNDED                 | REFUND_SUCCESS           |
| REFUNDED                 | NOTIFIED_CUSTOMER        | REFUND_COMPLETED         |
| COMPLETED                | NOTIFIED_CUSTOMER        | PAYMENT_SUCCESS          |
| REFUNDED                 | NOTIFIED_SECURITY        | FRAUD_DETECTED           |

## Описания состояний

- **CHARGED**: Деньги успешно списаны со счета клиента (CHARGE_CUSTOMER завершена). Транзакция готова к проверке.
- **CREDITING**: Идет перевод средств контрагенту (CREDIT_MERCHANT). Ожидание подтверждения.
- **COMPLETED**: Транзакция завершена успешно — деньги переведены, все проверки пройдены. Готово к уведомлению.
- **EXTERNAL_FRAUD_CHECK**: Выполняется внешняя антифрод-проверка (API внешних сервисов, с retry и таймаутами до 30 сек, max 3 попытки).
- **FAILED**: Транзакция провалилась необратимо (критическая ошибка, без возврата). Требует ручного анализа.
- **FRAUD_MANUAL**: Транзакция на ручной проверке (ожидание 20 мин от оператора или cut-off-time для default approve). Состояние ожидания.
- **INITIATED**: Платеж инициирован (INITIATE_PAYMENT), запись создана в БД, paymentId сгенерирован. Идемпотентно.
- **INTERNAL_FRAUD_CHECK**: Выполняется внутренняя антифрод-проверка (FraudCheck Service, правила и кэш в Redis).
- **NOTIFIED_CUSTOMER**: Клиент уведомлен о статусе (NOTIFY_CUSTOMER). Может быть параллельно с другими состояниями.
- **NOTIFIED_SECURITY**: Система безопасности уведомлена о подозрении (NOTIFY_SECURITY). Только при фроде/отказе.
- **REFUNDING**: Инициирован возврат средств (REFUND_CUSTOMER). Автоматическая компенсация.
- **REFUNDED**: Возврат средств клиенту завершен успешно. Транзакция закрыта с откатом статусов (REVERT_STATUS).