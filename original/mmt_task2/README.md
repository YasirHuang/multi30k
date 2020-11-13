This is the release of the dataset for the WMT 16 Multimodal
Translation Task 2: Crosslingual Image Description Generation.

The data is structured as follows:

'en'
  * 'train'
    - en_train.1, ... en_train.5
  * 'val'
    - en_val.1, ... , en_val.5

'de'
  * 'train'
    - 'de_train.1, ..., de_train.5'
  * 'val'
    - 'de_val.1', ..., de_val.5'

The 'en' and 'de' directories contain five descriptions per image for the 
29,000 training images and the 1,014 validation images.

See http://staff.fnwi.uva.nl/d.elliott/wmt16/splits.zip for details on
how the data was split. The order of the data in the 
(train|val)_images.txt file corresponds with the order of the descriptions.

Questions? d.elliott@uva.nl

1st March 2016.
