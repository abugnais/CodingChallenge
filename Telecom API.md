# Telecom API
Both api version and response format should be in the request headers

```php
<?php
/**
 * Roughly based on symfony
 */

/**
 * controller
 *
 * controller methods should only return data, data format and serialization should be done
 * using the format listener based on request Accept headers
 */
class TelecomController {

    /**
     * description: gets all phone numbers
     * endpoint:    GET /customers/phones/
     *
     */
    public function getAllPhoneNumbersAction() {
        return $this->entityManager->getRepository('PhoneNumber')->findAll();
    }

    /**
     * description: gets all phone numbers for the given customer
     * endpoint:    GET /customers/{id}/phones/
     *
     */
    public function getCustomerPhoneNumbersAction(Customer $customer) {
        return $customer->phoneNumbers;
    }

    /**
     * description: activates the given phone number
     * endpoint:    PATCH  /customers/phones/{id}/
     *
     */
    public function activatePhoneNumberAction(PhoneNumber $phoneNumber) {
        return $this->entityManager->getRepository('PhoneNumber')->activate();
    }
}

/**
 * Entities
 */
class Customer {
    private $id;

    private $name;

    /**
     * one to many
     * list of phone numbers for the current user
     */
    private $phoneNumbers;
}

class PhoneNumber {
    private $id;

    private $userId;

    private $value;

    private $is_active;

    /**
     * many to one
     * refers to the user
     */
    private $user;
}

/**
 * Custom Repository
 */
class PhoneNumbersRepository extends EntityRepository {
    public function activate(PhoneNumber $phoneNumber) {
        // might involve more business logic
        $phoneNumber->is_active =   1;
        $this->entityManager->persist($phoneNumber);
        $this->entityManager->flush();
        return $phoneNumber;
    }
}
```
