# Events to the rescue!

the problem with this state machine is that it publishes generic events, that needs to be parsed by every client that will listen to the event dispatcher, since it cannot be extended via inheritance, the best way to wrap it is by handling the event mapping logic in the dispatcher itself.

```php
<?php
/**
 * instead of extending StateMachine via inheritance, we can inject an event dispatcher
 * that will create more specific events, so that listeners wont need to parse event params
 * to figure out the meaning of an event
 *
 * usage example:
 *
 * $dispatcher  =   new StateMachinePropagator(
 *     [
 *         'out_for_delivary'    =>  [
 *             'recieved_at_hub'   =>  'shipment.reschedule',
 *             'delivered'         =>   'shipment.fullfilled',
 *         ],
 *         'ready_for_pickup'   =>  [
 *             'picked_up'      =>  'shipment.confirmend'
 *         ]
 *     ]
 * );
 *
 */
class StateMachinePropagator implements EventDispatcher {
    private $eventsStateMapping;

    private $listeners;

    public function __construct(array $eventsStateMapping) {
        $this->eventsStateMapping   =   $eventsStateMapping;
    }

    /**
     * event mapping + dispatching
     */
    public function dispatch($eventName, array $parameters = array()) {
        $aliasedEvent   =   $this->mapEvent($eventName, $parameters);

        if(isset($this->listeners[$aliasedEvent])) {
            foreach($this->listeners[$aliasedEvent] as $listener) {
                $listener($parameters);
            }
        }
    }

    /**
     * basic implementation
     */
    public function on($eventName, $calllable) {
        if(isset($this->listeners[$eventName])) {
            $this->listeners[$eventName][]  =   $calllable;
        } else {
            $this->listeners[$eventName]    =   array($calllable);
        }
    }

    /**
     * here we can map the generic event to a new more specific event based on parameters
     * @example ('state.change', ['from'=> 'shipped', 'to' => 'received']) -> 'order.return'
     * @param  string $eventName
     * @param  array  $parameters
     * @return string  new event name
     */
    private function mapEvent($eventName,  array $parameters) {
        $from   =   $parameters['from'];
        $to     =   $parameters['to'];
        return isset($this->eventsStateMapping[$from][$to]) ? $this->eventsStateMapping[$from][$to] : $eventName;
    }
}
```
