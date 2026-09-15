<!--
  @component Renders a settings page for security settings.
-->
<script lang="ts">
  import {globals} from '~/app/globals';
  import {ROUTE_DEFINITIONS} from '~/app/routing/routes';
  import Text from '~/app/ui/components/atoms/text/Text.svelte';
  import KeyValueList from '~/app/ui/components/molecules/key-value-list';
  import type {SecuritySettingsProps} from '~/app/ui/components/partials/settings/internal/security-settings/props';
  import {i18n} from '~/app/ui/i18n';
  import {createScreenshotPreventionState} from '~/common/utils/settings';

  const {services, actions}: SecuritySettingsProps = $props();

  const log = globals.unwrap().uiLogging.logger('ui.component.security-settings');

  const {router} = services;

  const workSettingsViewStore = $derived(services.settings.views.work);
  const privacySettingsViewStore = $derived(services.settings.views.privacy);

  const screenshotPreventionState = $derived(
    createScreenshotPreventionState($workSettingsViewStore, $privacySettingsViewStore, log),
  );

  const screenshotPreventionLabel = $derived(
    screenshotPreventionState.enabled
      ? $i18n.t(
          'settings--security.label--prevent-screenshots-enabled',
          'Taking screenshots is disabled',
        )
      : $i18n.t(
          'settings--security.label--prevent-screenshots-disabled',
          'Taking screenshots is enabled',
        ),
  );

  const hintText = $derived(
    screenshotPreventionState.mode === 'mdm'
      ? $i18n.t(
          'settings--security.hint--prevent-screenshots-mdm',
          'This setting is managed by your organization.',
        )
      : $i18n.t(
          'settings--security.hint--prevent-screenshots-unsupported',
          'This setting is not supported on your system.',
        ),
  );

  function handleClickChangePassword(): void {
    router.go({modal: ROUTE_DEFINITIONS.modal.changePassword.withoutParams()});
  }

  function handleScreenshotPreventionUpdate(enabled: boolean): void {
    actions.updateSettings({localScreenshotPrevention: enabled});
  }
</script>

{#snippet screenshotPreventionSwitch()}
  <KeyValueList.ItemWithSwitch
    key={$i18n.t('settings--security.label--prevent-screenshots', 'No screenshots')}
    disabled={screenshotPreventionState.mode !== 'changeable'}
    checked={screenshotPreventionState.enabled}
    hint={screenshotPreventionState.mode === 'changeable' ? undefined : hintText}
    onswitch={(state) => handleScreenshotPreventionUpdate(state.new)}
  >
    <Text text={screenshotPreventionLabel}></Text>
  </KeyValueList.ItemWithSwitch>
{/snippet}

<KeyValueList>
  <KeyValueList.Section
    title={$i18n.t('settings--security.label--app-protection', 'App Protection')}
  >
    <KeyValueList.ItemWithButton icon="edit" key="" onclick={handleClickChangePassword}>
      <Text text={$i18n.t('settings--security.label--change-password', 'Change App Password')} />
    </KeyValueList.ItemWithButton>
    {@render screenshotPreventionSwitch()}
  </KeyValueList.Section>
</KeyValueList>
