<script>
  import { createEventDispatcher } from "svelte";
  import {
    Row,
    Col,
    Tooltip,
    Button,
    Dialog,
    Card,
    CardTitle,
    CardText,
    CardActions,
  } from "svelte-materialify";
  import { AutoComplete } from "../";
  import { langs } from "../../utils/helpers/languages";

  let parsedLangs = [];
  langs.forEach((item) => {
    parsedLangs.push(item.name);
  });
  
  const dispatch = createEventDispatcher();

  export let languages_details;

  let imageSrc = "language.png";
  let active = false;

  function open() {
    active = true;
  }
  function close() {
    active = false;
  }

  function handleLanguage(customEvent, languageType) {
    languages_details[languageType].name = customEvent.detail.value;
    for (const lang of langs) {
      if (lang.name === customEvent.detail.value) {
        languages_details[languageType].shortName = lang.shortName;
      }
    }
  }

  function changeLanguages() {
    dispatch("languagesChanged", languages_details);
    close();
  }
</script>

<div
  on:mouseenter={() => (imageSrc = "languageWhite.png")}
  on:mouseleave={() => (imageSrc = "language.png")}
  class="image-button"
  on:click={open}
>
  <Tooltip left>
    <img src={imageSrc} alt="background image" />
    <span slot="tip">Select languages</span>
  </Tooltip>
</div>

<Dialog width="70vw" bind:active>
  <Card>
    <CardTitle>Select languages to use</CardTitle>
    <CardText style="height: 40vh">
      <Row>
        <Col>
          <AutoComplete
            inputDescription="Current Language"
            inputLabel="Language 1"
            inputPlaceholder="Select a language"
            data={parsedLangs}
            selectedValue={languages_details.current.name}
            on:changeValue={(customEvent) =>
              handleLanguage(customEvent, "current")}
          />
        </Col>
        <Col>
          <AutoComplete
            inputDescription="Second Language"
            inputLabel="Language 2"
            inputPlaceholder="Select a language"
            data={parsedLangs}
            selectedValue={languages_details.second.name}
            on:changeValue={(customEvent) =>
              handleLanguage(customEvent, "second")}
          />
        </Col>
      </Row>
    </CardText>
    <CardActions>
      <div class="card-action-wrapper">
        <Button on:click={close} text>Close</Button>
        <Button on:click={changeLanguages} text class="green-text">Save</Button>
      </div>
    </CardActions>
  </Card>
</Dialog>

<style>
  .image-button {
    position: absolute;
    top: 20px;
    right: 12px;
  }
  .image-button img {
    height: 50px;
    width: auto;
  }
  .card-action-wrapper {
    display: flex;
    width: 100%;
    justify-content: flex-end;
  }
</style>
