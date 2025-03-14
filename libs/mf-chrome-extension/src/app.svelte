<script lang="ts">
  import { onMount } from 'svelte';
  import { ConfigurationObject } from '@mf/core';

  import Modal, { getModal } from './core/components/modal.svelte'
  import { MFChromeExtensionActions } from './core/constant'
  import { configurations } from './stores/configurations-store';
  import { configurationSelected } from './stores/configuration-selected';
  import ConfigurationsTable from './components/configurations-table.svelte';
  import ConfigurationModal from './components/configuration-modal.svelte';
  import ConfigurationsByUriForm from './components/configurations-by-uri-form.svelte'

  onMount(() => {
    sendMessage(MFChromeExtensionActions.PopupOpened, null);
  });

  /**
   *
   */
  function onAddConfigurationDialog() {
    configurationSelected.set({
      selected: null,
      editable: false
    });

    getModal('configurationModal')?.open();
  }

  /**
   *
   */
  function onEditConfigurationDialog(e: CustomEvent) {
    const configuration = e.detail as ConfigurationObject;

    configurationSelected.set({
      selected: configuration,
      editable: true
    });

    getModal('configurationModal')?.open();
  }

  /**
   *
   */
  function onSubmitConfiguration(e: CustomEvent) {
    const configuration = e.detail as ConfigurationObject;

    const { selected, editable } = $configurationSelected;

    getModal('configurationModal')?.close();

    if (editable) {
      const editedConfiguration = {
        ...selected,
        ...configuration
      };

      sendMessage(MFChromeExtensionActions.UpdateConfigurationObject, editedConfiguration);
      return;
    }

    sendMessage(MFChromeExtensionActions.AddConfigurationObject, configuration);
  }

  /**
   *
   */
  function onSubmitConfigurationsByUriDialog() {
   getModal('configurationsByUriModal')?.open();
  }

  /**
   *
   */
  function onSubmitConfigurationsByUri(e: CustomEvent) {
    const uri = e.detail as ConfigurationObject;

    getModal('configurationsByUriModal')?.close();
    sendMessage(MFChromeExtensionActions.AddConfigurationObjectsByUri, uri);
  }

  /**
   *
   */
   function onCloneConfiguration(e: CustomEvent) {
    const configuration = e.detail as ConfigurationObject;
    sendMessage(MFChromeExtensionActions.CloneConfigurationObject, configuration);
  }

  /**
   *
   */
  function onToggleActiveConfiguration(e: CustomEvent) {
    const configuration = e.detail as ConfigurationObject;
    sendMessage(MFChromeExtensionActions.SwitchConfigurationObject, configuration);
  }

  /**
   *
   */
  function onRefreshPage() {
    chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
      chrome.tabs.reload(tabs[0].id);
    });
  }

  /**
   *
   */
  function onClosePopup() {
    sendMessage(MFChromeExtensionActions.ClosePopup, null);
  }

  /**
   * Sends the json object to the content script
   */
  function sendMessage(action: MFChromeExtensionActions, payload: any) {
    window.parent.postMessage({ action, payload }, '*');
  }

</script>

<main>
  <Modal id="configurationModal">
    <ConfigurationModal on:submit={onSubmitConfiguration}/>
  </Modal>

  <Modal id="configurationsByUriModal">
    <ConfigurationsByUriForm on:submit={onSubmitConfigurationsByUri} />
  </Modal>

  <ConfigurationsTable {configurations}
                       on:add={onAddConfigurationDialog}
                       on:addByUri={onSubmitConfigurationsByUriDialog}
                       on:clone={onCloneConfiguration}
                       on:edit={onEditConfigurationDialog}
                       on:toggleActive={onToggleActiveConfiguration}
                       on:refresh={onRefreshPage}
                       on:close={onClosePopup}
  />
</main>

<style lang="scss">
  main {
    min-height: 100%;
    padding: 10px;
    overflow: hidden;
  }
</style>
