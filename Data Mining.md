# Data Mining

simplest way to write a migration tool is by implementing the ETL pattern, this will require 3 main interfaces: 
Readable datasource, Writable Datasource, Transformation function, I wrapped the whole thing in one class that 
will wire up the whole process

```php
<?php
namespace ETL;

class Worker {
    public function __construct(Transformer $transformer, ReadableDatasource $from, WritableDatasource $to) {
        $this->transformer  =   $transformer;
        $this->from         =   $from;
        $this->to           =   $to;
    }

    public function run() {
        $data           = $this->extract();
        $preparedData   = $this->transform($data);
        $this->load($preparedData);
    }

    private function extract() {
        $this->from->connect();
        $data   =   $this->from->read();
        $this->from->disconnect();
        return $data;
    }

    private function transform(array $data) {
        return $this->transformer->transform($data);
    }

    private function load(array $data) {
        $this->to->connect();
        $this->to->write($data);
        $this->to->disconnect();
        return true;
    }
}

interface Datasource {
    public function __construct(array $connectionParams);

    public function connect();

    public function disconnect();
}

interface ReadableDatasource extends Datasource {
    public function read(array $params);
}

interface WritableDatasource extends Datasource {
    public function write(array $params);
}

interface Transformer {
    public function transform(array $params);
}
```
