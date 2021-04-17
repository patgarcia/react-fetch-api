# react-fetch-api
Semantical options when using .fetch inside a react functional component

### Scenario Context
Let's assume we want to use the [MixCloud API](https://www.mixcloud.com/developers/) to create a functional component that, given an artist name passed as a prop, calls the API and performs some opeartions. Like getting an image or loading a track in their footer player.

__Assumption:__ The artist name can include spaces.
```jsx
<Tribute artist={ props.name } />
```

### Simple case: Load full response in a state variable

The simple case shows how to call the API using ```fetch, useEffect and useState```. It assigns the entire response JSON object to the state variable ```data```, which may force us to use long access paths to the information. i.e. ```data.data[0].pictures['640wx640h']```

```jsx
import React, { useEffect, useState } from 'react';
    
    const Tribute = (props) => {
        const [data, setData] = useState(null);

        useEffect(() => {
            fetch(`https://api.mixcloud.com/search/?q=${props.artist.split(' ').join('+')}&type=cloudcast`)
                                .then(res => res.json())
                                .then(res => setData(res))
        } , [props])


    return (
        <div className="tribute">
        <h2>{ data ? data.data[0].name : 'fetching' }</h2>
        <button data-mixcloud-play-button={ data ? data.data[0].key : 'fetching' } >{ data ? "▶ Play" + data.data[0].name : 'fetching' }</button>
        <img alt={ data ? data.data[0].name : 'fetching' } src={ data ? data.data[0].pictures['640wx640h'] : 'fetching' }></img>
    </div>
    );
};

export default Tribute;
```

### Advanced case: Destructuring response in descriptive variables saved as properties in a state object
Adding ```useCallback``` to the mix would allows us to perform a destructuring assignment to the response, later using these variable to create an object with descriptive properties that helps us access our information in a cleaner manner.
```jsx
import React, { useEffect, useState, useCallback } from 'react';
    
    const Tribute = (props) => {
        const [data, setData] = useState(null);
        const fetchAPI = useCallback(async () => {
                let artist = props.artist;
                artist = artist.split(' ').join('+');
                let res = await fetch(`https://api.mixcloud.com/search/?q=${artist}&type=cloudcast`)
                                    .then(res => res.json())
                                    .then(res => res)
                const { name, key, url, pictures } = res.data[0]
                const id = name.split(' ').join('')
                setData({ name, key, url, pictures, id })
            } , [props]);

        useEffect(() => {
            fetchAPI()
        }, [fetchAPI])


    return (
        <div className="tribute">
            <h2>{ data ? data.name : 'fetching' }</h2>
            <button data-mixcloud-play-button={ data ? data.key : 'fetching' } >{ data ? "▶ Play" + data.name : 'fetching' }</button>
            <img alt={ data ? data.name : 'fetching' } src={ data ? data.pictures['640wx640h'] : 'fetching' }></img>
        </div>
    );
};

export default Tribute;
```
