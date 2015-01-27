# Telecom API
- Both API version and response format should be in the request headers
- Persistance and business logic are encapsulated inside the service
- This is roughly based on symfony

```php
<?php
/**
 * 
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
        return $this->get('phoneNumber.service')->findAll();
    }

    /**
     * description: gets all phone numbers for the given customer
     * endpoint:    GET /customers/{id}/phones/
     *
     */
    public function getCustomerPhoneNumbersAction($id) {
        return $this->get('phoneNumber.service')->findByCustomerId($id);
    }

    /**
     * description: activates the given phone number
     * endpoint:    PATCH  /customers/phones/{id}/
     *
     */
    public function activatePhoneNumberAction($id) {
        return $this->get('phoneNumber.service')->activate($id);
    }
}

class PhoneNumberService {
    private $em;

    public function __construct(EntityManager $em) {
        $this->em = $em;
    }

    public function findAll() {
        return $this->em->getRepository('PhoneNumber')->findAll();
    }

    public function activate($id) {
        $phoneNumber    =   $this->em->getRepository('PhoneNumber')->find($id);
        $phoneNumber->setIsActive(1);
        $this->em->persist($phoneNumber);
        $this->em->flush();
        return $phoneNumber;
    }

    public function findByCustomerId($customerId) {
        return $this->em->getRepository('PhoneNumber')->findBy(array('customer_id'=>$customerId));
    }
}
```
