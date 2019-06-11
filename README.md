# Entity-address dataset for 2010-2018 Bitcoin transactions


The dataset consists of categorical entity labels and associated public key Bitcoin wallets as scraped from the [Wallet Explorer] (https://www.walletexplorer.com/) website in April 2018. The dataset also contains [BlockSci] (https://github.com/citp/BlockSci) internal identifiers  exported from BlockSci v0.4.5 for fast manipulation of the associated Blockchain data using BlockSci.


We release this dataset with the intent of facilitating further research on Bitcoin data requiring entity labels. 


This dataset made available here, together with the full 2010-2018 Bitcoin Blockchain graph in BlockSci format (which can be found on BlockSci, was used in the paper [*Characterizing Entities in the Bitcoin Blockchain*] (https://arxiv.org/abs/1810.11956), presented at the IEEE International Conference on Data Mining 2018 as well as in the paper [*A Probabilistic Model of the Bitcoin Blockchain*] (https://arxiv.org/abs/1812.05451). If you use this dataset, please cite the papers

```bibtex
@InProceedings{JBWD_ICDM18,
  author       = {Jourdan, Marc and Blandin, Sebastien and Wynter, Laura and Deshpande, Pralhad},
  title        = {Characterizing entities in the Bitcoin Blockchain},
  booktitle    = {Data Mining Workshop (ICDMW), 2018 IEEE International Conference on},
  year         = {2018},
  pages        = {--},
  organization = {IEEE}
}

@InProceedings{JBWD_CVPR19,
  author    = {Marc Jourdan and
               Sebastien Blandin and
               Laura Wynter and
               Pralhad Deshpande},
  title     = {A Probabilistic Model of the Bitcoin Blockchain},
  booktitle = {Computer Vision and Pattern Recognition Workshop (CVPRW), 2019},
  pages     = {--},
  organization = {IEEE}
}
```

In the absence of BlockSci data, this dataset can be used to analyze the distribution of distinct addresses across distinct entities, by entity type. More comprehensive studies using the full Blockchain graph, for instance from BlockSci, can also be conducted without having to scrape entity-address mapping, which can be cumbersome.


## Downloading the Dataset


The current zipped version of the dataset is available [here](https://polybox.ethz.ch/index.php/s/GUEFVnOEPrMxY2l/download). It is 1 GB and contains 30.331.700 addresses.


### Methodology


The entity-address mapping was obtained in April 2018 by scraping [WalletExplorer](https://www.walletexplorer.com/). Five categories of interest (Exchange, Mining Pool, Gambling, Services, Historical) were scraped as per Wallet Explorer. Adresses from the sections "old" weren't scrapped.


The scraped dataset was enhanced using the multi-input (also known as common spending) clustering heuristic. This heuristic groups addresses which are inputs of the same transaction (see the paper and references therein for more details). We used BlockSci for interfacing with the Blockchain in order to recursively enhance the clusters. We ran it on the Blockchain of height below 514.9711, corresponding to blocks created before March 24th 2018, 15:19:02, which contains about 500.000.000 addresses.


## Dataset Description


The zipped folder contains 5 .CSV files: Exchanges_full_detailed.csv, Gambling_full_detailed.csv, Historic_full_detailed.csv, Mining_full_detailed.csv, Services_full_detailed.csv. 


The files include the addresses associated with Exchanges, Gambling services, Mining Pools, other Services and Historical services (services that are no longer active).


Each CSV file has 5 columns:
- hashAdd : string - Public key of the address.
- date_first_tx : pandas.Datetime - Date of the first known transaction involving this address.
- {exchange, gambling, historic, mining, service} : string - Name of the associated entity.
- add_type : int - internal BlockSci type of address.
- add_num : int - internal BlockSci identifier for the given type of address. The tuple (add_type,add_num) is equivalent to the hashAdd, however it speeds the access to the representation of the address in BlockSci.


To plug the address type (in integer) to the BlockSci implementation, we used the following dictionary to retrieve the internal representation of an address via `address_from_index` . This function requires the type of the address, an address_type object from blocksci_interface, stored as an integer and the actual id which is the stored integer. For example, the following script provides the cluster of BlockSci addresses associated to the addresses of entities within the Exchanges_full_detailed.csv file.

```python
import pandas as pd 
import blocksci
from blocksci.blocksci_interface import address_type

chain = blocksci.Blockchain("your_bitcoin_parsed_path")

add_type_dict = {
    address_type.pubkey : 0,
    address_type.multisig_pubkey : 1,
    address_type.multisig : 2,
    address_type.nonstandard : 3,
    address_type.nulldata : 4,
    address_type.pubkeyhash : 5,
    address_type.scripthash : 6,
    address_type.witness_pubkeyhash : 7,
    address_type.witness_scripthash : 8
}

df_exchanges = pd.read_csv("path_to_addresses/Exchanges_full_detailed.csv")


cluster_addresses = dict()
num_list = list(df_exchanges['add_num'])
type_list = list(df_exchanges['add_type'])
exc_list = list(df_exchanges['exchange'])
exc_set = set(exc_list)

for exc in set(exc_list):
    cluster_addresses[exc] = []
    
for n,t,exc in zip(num_list,type_list,exc_list):
    address = chain.address_from_index(n,add_type_dict_rev[t])
    cluster_addresses[exc].append(address)
    
```


**Remark:** The last two parameters can be removed if not using BlockSci. Moreover they are application specific, they must be recomputed with your own BlockSci parsing of the Blockchain.


### Further questions

In case of any problem, please contact me at jourdanm@student.ethz.ch
