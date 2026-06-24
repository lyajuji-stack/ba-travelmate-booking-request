This diagram shows the interaction between the User, UI, Search Service,
and Hotel Database during the hotel search, filtering, and selection flow.
It covers the main flow and alternative scenarios (invalid dates,
no hotels found, empty filter results).

```mermaid
sequenceDiagram
actor User as User (Guest)
participant UI as UI (Search Page)
participant SS as Search Service
participant DB as Hotel Database

User->>UI: enterDestination(city / hotel name)
User->>UI: selectDates(checkIn, checkOut)

alt checkOut <= checkIn (Alt 2a: Invalid Date Range)
UI-->>User: blockInvalidSelection()
UI-->>User: showError("Check-out must be after check-in date")
UI-->>User: promptCorrectDates()
else valid date range
User->>UI: specifyGuestsAndRooms(adults, rooms)
User->>UI: clickSearch()
UI->>SS: validateAndSearch(destination, dates, guests, rooms)
SS->>DB: queryHotels(params)

alt no hotels match (Alt 6a: No Hotels Found)
DB-->>SS: returnEmpty()
SS-->>UI: showNoHotelsFound() + suggestAlternatives()
UI-->>User: promptModifySearch()
else hotels found
DB-->>SS: returnHotelList(hotels[])
SS-->>UI: returnSearchResults(hotels[])
UI-->>User: displayHotelList(sorting="Recommended")

opt user wants to filter (Alt 7a)
User->>UI: applyFilters(price, amenities, rating)
UI->>SS: filterHotels(activeFilters)

alt no results match filters
SS-->>UI: showNoResultsMatchFilters()
UI-->>User: showClearFiltersOption()
User->>UI: clickClearFilters()
UI-->>User: restoreOriginalList()
else results found
SS-->>UI: returnFilteredList(filteredHotels[])
UI-->>User: updateHotelListRealTime(filteredHotels[])
end
end

User->>UI: clickHotelCard(hotelId)
UI->>SS: getHotelDetails(hotelId)
SS->>DB: fetchHotelDetails(hotelId)
DB-->>SS: returnHotelDetails(details)
SS-->>UI: returnHotelDetails(details)
UI-->>User: redirectToHotelDetailsPage(hotelId)
UI->>SS: saveSearchParamsToSession(destination, dates, guests)
end
end
```
