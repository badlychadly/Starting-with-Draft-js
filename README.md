# Starting with-Draft-js

For the past Couple of months I have been testing out solutions for client side text editing. I have tried a few different wysiwyg frameworks for react but I have finally found one that I can really appreciate called Draft-js. Draft-js is what Facebook uses when you write a comment or a status. It is how they are able to do things like tagging friends when you are typing out their name. In this article I want to show you how to get Draft-js up and running in your React application. Check out the docs (here)[https://draftjs.org/docs/getting-started].

For installation run 
```
npm install draft-js react react-dom
# or alternately
yarn add draft-js react react-dom
```

## Basic Setup
On the Docs you can copy and paste their example to get get started
```
import React from 'react';
import ReactDOM from 'react-dom';
import {Editor, EditorState} from 'draft-js';

class MyEditor extends React.Component {
  constructor(props) {
    super(props);
    this.state = {editorState: EditorState.createEmpty()};
    this.onChange = (editorState) => this.setState({editorState});
  }
  render() {
    return (
        <Editor editorState={this.state.editorState} onChange={this.onChange} />
    );
  }
}
```
This will enable you to write text in the Browser but not much else yet. In the constructor we set state to equal an empty `EditorState`, then we have `onChange` Which takes the `EditorState` as an argument and sets the local state equal to it. This seems confusing at first but the thing to understand is that the Draft model is immutable. Inside this model there is an object called `EditorState` which has all the data about the editor being rendered to the browser. Then inside the `EditorState` we can find more objects but the two most important are `ContentState` and `SelectionState`. The `SelectionState` object contains data about the cursor position and focus, While the `ContenetState` contains the `ContantBlocks` objects that contain all the data relevent to the actual text in the editor, like inlineStyleRanges and all the blocks that make up all the text in the editor. All the text and styles are tracked by the `ContentBlock` objects, which tell us if a block is `h1` or `p` etc. These blocks are tracked with a unique key to tell the editor how the content is structured. 


### Updating EditorState
All the contents within EditorState are immutable so the way to retrieve and alter data is limited. In the Draft documentation you can find instance methods to retrieve data, for example to get SelectionState from EditorState you use the method `getSelection()` which returns a SelectionState Record. For ContentState you use `getCurrentContent()` which returns a ContentStateRecord. Assuming you are using the set up above it would look something like this:
```
this.state.editorState.getSelection()
##returns SelectionState {_map: Map, __ownerID: undefined}

this.state.editorState.getCurrentContent()
##returns ContentState {_map: Map}
```
If you want to alter the EditorState for some kind of custom feature depending on if you change ContentState or SelectionState you have you make a copy of them and use `.merge()` to merge the changes in the copy that you want, then depending on Which state you alter you need to use the appropriate method to combine your new object with the EditorState. Finally we can pass the new EditorState instance to an `onChange()` method to update the local state. 

For example I made some changes to SelectionState because when I would click on a button in my toobar the editor would loose focus. To handle this I added an `onClick` event to the parent `div` and passed a function with the following code:
```
    const { editorState } = this.state;
    
        ###get copy of current SelectionState
    const selectionState = editorState.getSelection()
    
        ###merge Changes you want
      const newSelection = selectionState.merge({
        hasFocus: true
      })
      e.stopPropagation()
      
          ###Combine new SelectionState to EditorState
      const newEditorState = EditorState.forceSelection(editorState, newSelection);
      
      ##pass to onChange
      this.onChange(newEditorState)

```
Now when I click anywhere inside the parent `div` the cursor will stay in the correct position and won't lose focus.


Draft-js is super powerful tool and we haven't even began to cover Rich Utils or Decorators. There is plenty more to do and learn with Draft-js i'm excited to keep using it.
