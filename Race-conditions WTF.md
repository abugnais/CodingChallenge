# Race-conditions WTF

Scenario:
Jhon downloaded todays version of the document and he started adding to it, later Frank downloaded the same version of this document and started editing, John finished working for the day and uploaded the new version of the document, then Frank who has a document that has his changes, but not John's uploaded the document that he has, so John's changes were overwritten, the same thing happend with Mark and Alex.

Solution:
Versioning, multiple options are available:

1. using a version control system at the back-end (too complicated when dealing with compressed content like odt).

2. storing all historical versions of the same document (ineffecient, not feasible).

3. storing the downloaded document version number/ user-id pair and warning the user when uploading an older version of the document.

option #3 seems to be the simplest, if a user tries to upload an older verison of the document then the system will warn him and ask for confirmation.

```php
<?php
class Document {
    protected $id;

    protected $version_number;

    protected $file;

    //...the rest of the fields
    public function prePersist() {
        // increment document version
        $this->version_number += 1;
    }
}

class User {
    protected $id;

    //...the rest of the fields
}

class DocumentUserVersion {
    //-- Unique
    protected $user_id;

    protected $document_id;

    protected $version_number;
}

class DocumentService {
    public function download($user_id, $document_id) {
        $document               =   $this->em->getRepository('Document')->find($document_id);
        $documentUserVersion    =   new DocumentUserVersion();
        $documentUserVersion->setUserId($user_id);
        $documentUserVersion->setDocumentId($document_id);
        $documentUserVersion->setVersionNumber($document->version_number);

        $this->em->persist($documentUserVersion);
        $this->em->flush();
        return $document;
    }

    public function upload($user_id, $document_id, $file) {
        $document   =   $this->em->getRepository('Document')->find($document_id);
        $documentUserVersion    =   $this->em->getRepository('DocumentUserVersion')->findBy(array('user_id'=>$user_id, 'document_id'=>$document_id));
        if(!empty($documentUserVersion) && $documentUserVersion->version_number < $document->version_numberfo) {
            throw SomeException('warn user of a possible overwrite');
        }

        $document->setFile($file);
        $this->em->persist($document);

        $documentUserVersion->setVersionNumber($document->getVersionNumber());
        $documentUserVersion->setUserId($user_id);
        $documentUserVersion->setDocumentId($document_id);
        $this->em->persist($documentUserVersion);

        $this->em->flush();
        return $document;
    }
}

```


