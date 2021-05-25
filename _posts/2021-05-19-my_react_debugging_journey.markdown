---
layout: post
title:      "My React Debugging Journey"
date:       2021-05-19 02:34:24 -0400
permalink:  my_react_debugging_journey
---

This React project was the most challenging of all the projects and was quite the project to end my coding journey with Flatiron (there is still more, career services and coding interviews still to come even after graduation!).

I encountered plenty of roadblocks, from forgetting how to send nested objects to Rails (that was a fun one to debug), to setting up my routes if I wanted it under a specific path. This post will outline some of the problems I encountered and what I did to solve them, but this is by no means an exhaustive list because there would not have been much coding had I stopped to write about every issue I faced! Though I encountered just as many Rails issues as I did with React, I will focus more on the React side of my debugging adventures.


## Editing number of breaths in the edit/new sequence form

A value in an uncontrolled input element can only hold an initial value. If one needs that value to be updated, a kind of hack is to specify the key in the input of the value that needs to be updated, despite it not really needing a key - in order to make it unique. This was what happened when I needed the default value of my input for number of breaths to be what the pose was initally set to. `<input ...defaultValue={this.props.poseInseq}>` Whenever I tried to change the value in the input, it refused to update and show the value. I then had to add a key of some sort: `<input key={this.props.poseInSeq.num_breaths}` in order for it to display what I had entered. This one stumped me for a long time as I wouldn't have known this was the behaviour had I not googled to understand what was happening.

    src/components/sequences/SeqPoseDraggableEdit.js

```
                    <input key={this.props.poseInSeq.num_breaths}
                        type="text"
                        defaultValue={this.props.poseInSeq.num_breaths}
                        name="num_breaths"
                        onChange={(event) =>this.props.onChange(event)}
                        onBlur={(event) => this.props.onBlur(event, this.props.index)}>
                    </input>
```

## Anti-Pattern

Constantly passing down props needed in an initial state will cause what is an "anti-pattern" if you don't pay attention carefully how you have set your states and your props. What can happen sometimes is the "source of truth" is now confused since I was accessing both props and state and trying to set the state with props (because I was confused what was what despite using Redux). This caused the state have one source of truth as I was modifying it when I shouldn't have. I don't have a good example as it was more a case of cleaning up my states and props to eliminate the errors.


## Displaying the poses in sequence did not show the latest changes after updating sequence.

This one stumped me for a long time. Whenever I edited a sequence, specifically removing or deleting poses to the sequence in the edit form (`<SeqForm>`), when I went to view the modified sequence under `sequences/:id`, the poses would not update when I wanted to display the sequence (`<SeqShow>`). Instead, it would show the previous state and I was confused as to what was going on. There were several issues here.

One, I discovered that after retrieving the sequence that had been modified, I didn't actually update the sequences state in the sequence reducer when `EDIT_SEQUENCE` was dispatched, which meant that when I went to view it, the state wasn't being updated correctly despite getting it from the server. It wasn't until I clicked on another component/route would it retrieve the sequences from the servers with the sequence and its updated poses. This was because I had handled returning the correct state in all other action types for the sequence reducer but `EDIT_SEQUENCE`. In addition, I noticed that submission of the sequence after an edit was submitting the `created_at`, `updated_at` information that I did not want to include. This meant specifying exactly the keys I wanted to include in `pose_in_sequences` from the controller.

Two, unfortunately, the above still didn't solve my problem. I had set it up where, after a dispatch to edit a sequence, the route would immediately display and forward to displaying the sequence list. I found that if I was impatient and chose to view the sequence I had just updated, what would happen is the sequence had not completely finished updating on the server yet, which meant I was served an older view of the sequence. When a props is updated using redux, it should automatically refresh the page with the updated data. But, it was not updating. I had to investigate why.

In my `<SeqShow>` component where I display all the details of my sequence, the only redux state is for all sequences, not for one sequence. After the `<SeqShow>` component is mounted, the sequence is retrieved from the id using  `props.match.id` and then set to `state.sequence`. However, since the sequence is only a local state, once the redux sequences prop is updated, I was not updating the sequence as I was showing the pose based on the local sequence state, and not the props. Therefore the sequence remained the same. This was a bit tricky to catch, partially due to the messiness of my code since I had passed the state to another function but named it data, so I hadn't realized I was using `state.sequence` as opposed to the sequence derived from props (which I would have to retrieve again).

However, all the above was slightly moot when I started checking whether or not the correct data was being loaded after accessing the pages individually.  It did not, which required a slight redesign.

## Refreshing and Accessing Individual Pages

After struggling with reloading of a `<SeqShow>` component or `<SeqForm>` with edit component that was derived from iterating through `props.sequence` and not through a server call. It was suggested to me by the section lead (Dakota) to create the server action of `getSequence` to retrieve the sequence I needed when the page matched the `sequences/:id` and when the `props sequences` were empty. This made it more efficient so that I would only do an additional server call to retrieve the sequence in the necessary scenario as opposed to always when I already have the sequence props. I had to use `getDerivedStateFromProps` to make sure that the sequence state was updated when the `props.sequence` was available, otherwise the state would never get updated with the sequence information, if the state was set prior to the props.sequence getting updated.

Arming myself with this knowledge, I also re-worked on my `<SeqForm>` edit portion of the component. This was another scenario in which I needed to set an initial state based on a specific sequence when a user either refreshed or was given a direct link because it would disappear upon reload. 

## Routes

I ran into the problem of `/sequences/:id` matching with `/sequences/add` in my form script so it was always trying to retrieve a sequence even though I was under the view of just wanting to add a new sequence. This could be avoided by putting the `<Switch>` router tag in `App.js`, but only after I spent a ton of time trying to solidify all the various `getDerivedStateFromProps` functions of several of the components (`<SeqShow>`,`< SeqForm>`). Lastly and **MOST IMPORTANTLY**, I found the order of the **Routes**  was what fixed it and that `/add` path needed to be prior to the `/:id` path otherwise it would always fall into the `<SeqShow>` path.


## Error handling.

I needed a way to display errors, and initially had started with handling displaying error messages component by component. This caused the code to be immediately more messy and I didn't like how if I were to grow my application I would have to have a local errors state in each component required for rendering.

I did discover that I could set an error message  that was fairly controlled by setting all my actions to return a Promise that could be caught within the components and set a local error state with whatever string I set. This was a valid method in handling errors but I decided upon using an **errorReducer** that would handle all form handling errors in addition to authentication/unauthorized errors. I created a function that would be called whenever the status returned from the server was other than `OK` or `200` (`handleServerError`). From there, it would dispatch an `'ERROR'`, or a `'NOT_AUTHENTICATED'` error depending on the code status. Then wherever I had placed the `<Error>` presentational component, the component would display the error. I liked this solution best, that way the error can be controlled through the global state, vs the local component state which can get cumbersome to implement as an application scales.

    src/actions/errors.js

```
export const handleServerError = (response, dispatch) => {
    return response.json().then(json => {
        dispatch({type: 'ERROR', error: json.error})
        if (response.status === 401)
            return Promise.reject(dispatch({type: NOT_AUTHENTICATED}))
        else 
        
            return Promise.reject(dispatch({type: 'ERROR', error: json.error}))
        })

}
```

## Corner Case Testing

I noticed that if I switched between users, I would see the previous user's data flash on my screen before loading the current user's state which is a definite no no. By dispatching a '`CLEAR_STORE`' in the `logout` and `withAuth` action methods that set the state back to the intial state in each of the reducers, I was able to fix this issue.

```src/reducers/sequences.js```
		
```

...reducer...
        case 'CLEAR_STORE':
            return {
                ...state,
                sequences: [],
                selSequence: {},
                requesting: false
            }
...reducer etc...
```

Each of these above bugs took a long time to debug, but with every bug I was able to learn more about how React is structured. I still have to sit and pause everytime I am faced with a problem in my application, but knowing the steps in which things are loaded, the life cycles in which data is retrieved and displayed makes debugging much quicker than it had initially. Stopping to pause and think before frantically throwing the kitchen sink at a problem allowed me to understand React better and debug quicker as I got more comfortable with the framework. There is still so much to learn about React (such as the newest hooks) and modifications I can do to improve my application, but now I am confident that I have the skills required to continue to learn and improve upon what I have created.



