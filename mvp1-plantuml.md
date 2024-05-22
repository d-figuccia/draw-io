<div hidden>
```
@startuml mvp1
title MVP1

legend right
    |<#9AB9FF>     | API Call (synch)|
    |<#D0B6E1>     | Event-driven communication (asynch)|
endlegend

package "3rd-parties" {
  component GoogleGeocodingAPI
}

cloud "SAP-Cloud-Commerce\n\n" {
  component Hybris
  component ProxyPass
  database AzureStorage
}

package "Frontend-SPA" {
  component StoreLocatorComponent
  component StoreDetailsComponent
  component AllStoresComponent
}

package "Microservices (Oldgen)" {
  component ImpexGen
  component yCatalog
  folder Queues {
    queue RabbitMQ1
    queue RabbitMQ2
  }
}

cloud "Nextgen" {
  package "BE-Nextgen" {
    component StoreServiceMVP1 {
      [API \n1. getNearestStores \n2. getStore \n3. getAllStoresSortedByTown \n4. getStoresByNames \n5. createStore \n6. updateStore \n7. deleteStore\n... \n\nConsumer \n- loadStoresMessage\n(call yCatalog API getStoreByName\nfor each store in the load message)]
    }
  }
  package "FE-Nextgen" {
    component NextGenBackofficeMVP1
  }
}

StoreLocatorComponent --[#9AB9FF]> ProxyPass: getNearestStores\n
StoreDetailsComponent --[#9AB9FF]> ProxyPass: getStore\n
AllStoresComponent --[#9AB9FF]> ProxyPass: getNearestStores\n
ProxyPass --[#9AB9FF]> Hybris : \n\n\n\nNO\n(proxy pass\ndeactivated)
ProxyPass --[#9AB9FF]> StoreServiceMVP1 : YES\n(proxy pass\nactivated)\n\n\n\n
StoreServiceMVP1 --[#9AB9FF]> GoogleGeocodingAPI : geocodeAddress
NextGenBackofficeMVP1 --[#9AB9FF]> StoreServiceMVP1 : 1. createStore\n2. updateStore\n3. deleteStore\n...
StoreServiceMVP1 --[#9AB9FF]> yCatalog : " " "getStoreByName\n(called for each store\nin the load message)"
ImpexGen --[#9AB9FF]> StoreServiceMVP1 : getStoresByNames\n(called ones for all the stores\nreceived in the load message)
StoreServiceMVP1 --[#D0B6E1]> RabbitMQ1 : produce loadStoresMessage\n(a replica of the one\nreceived from the yCatalog)\n\n\n\n\n\n\n
RabbitMQ1 --[#D0B6E1]> ImpexGen : consume loadStoresMessage
yCatalog --[#D0B6E1]> RabbitMQ2 : produce loadStoresMessage
RabbitMQ2 --[#D0B6E1]> StoreServiceMVP1 : consume loadStoresMessage
ImpexGen --[#D0B6E1]> AzureStorage
AzureStorage --[#D0B6E1]> Hybris

@enduml
```
</div>

![](mvp1.svg)
