# [Refactor] domainFileProvidersUsableStatus

### What has been done?
- Create _add_attachments_button_fragment_defs.ts_, which uses scv _domainFileProvidersUsableStatus_
  - `const FileProviderUsableStatus: ComputedObjectFragmentDef`
  -  `const Domain: DbObjectFragmentDef`   
- Create _upload_attachments_button_fragment_defs.ts_, which calls into _add_attachments_button_fragment_defs.ts_ 
  - `const Domain: DbObjectFragmentDef`
  - `const DomainUser_current: DbObjectFragmentDef`
- Enable _file_provider_usable_status_def.ts_, set
  - Set `.deployedToLunaDb()` for _file_provider_id_ and _usable_status_ fields
  - Set `deployedToLunaDb: true,`
- Identify all the call sites of _upload_attachments_button.ts_, and make sure that:
  - `domain` is being passed down when creating `UploadAttachmentsButton` instead of `domainId` (Talked with Robyn that the `domain` object is needed for scv `domainFileProvidersUsableStatus`)
  - Retrieve `domain` object from `this.props.currentDomainUser.domain` if possible
  - `UploadAttachmentsButtonFragmentDefs.DomainUser_current` is being included in the call sites' fragement def
  - List of call sites need to be updated:
    - [project_overview_key_resources_empty_state.ts](https://github.com/Asana/codez/pull/131619/files#diff-611ab1d7663d05663280d03e04667d3e180dd9cc3db67bc1b60c81592bfc7d6eR50)
    - [quick_add_task_toolbar.ts](https://github.com/Asana/codez/pull/131619/files#diff-e3278f0919d8f567279bff81b788a6811e1aeff98cf976f80ef3c9db394680f8R184)
    - [status_report_creator_header.ts](https://github.com/Asana/codez/pull/131619/files#diff-3e20cfef352fa14bb851d61edaaa336484efdbd49a9d3812b0128a10d7bd0b5aR634)
    - [composer_attachments_pile.ts](https://github.com/Asana/codez/pull/131619/files#diff-0f746ba60a592770ac2148bf2320ef65d75eabc73f50ff4eb96d66ed1dc07708R251)
    - [task_attachments.ts](https://github.com/Asana/codez/pull/131619/files#diff-075dc3d2c9c203e6dc605bf452f6fe682383fd2ed76d5ed5bc9bae0af8325b26R113)
    - [comment_composer.ts](https://github.com/Asana/codez/pull/131619/files#diff-57b69e00414700531caa85ac346bd53df818693987bf91eac850f07eddbd92e7R375)
    - [conversation_composer.ts](https://github.com/Asana/codez/pull/131619/files#diff-50ef85744bd2d139335644734e3d3b9bd74accfbdaebd9460c57beae7eca864dR6)
    - [single_task_pane_toolbar.ts](https://github.com/Asana/codez/pull/131619/files#diff-f05a0dd8b30185eea45cf7946d3c02aa354962a4d20c146b958df23271cf4e4fR340)
    - #TODO: [internal_attachments_piles.ts](https://github.com/Asana/codez/blob/next-master/asana2/asana/internal_apps/components/internal_attachments_pile.ts#L171) 

## What need to be done?
  - For [internal_attachments_piles.ts](https://github.com/Asana/codez/blob/next-master/asana2/asana/internal_apps/components/internal_attachments_pile.ts#L171), the `currentDomainUser` is not available to use
  - Traced multiple layers up and realized that we might need a `LoadingBoundary`, looked into example using `ApprovedOrBlockedAppsListBoundary`
  ```
  -> internal_attachments_piles.ts
  -> cue_builder_asset_uploader
    -> cue_builder_modal_settings
      -> cue_builder_settings_and_preview
        -> cue_builder_cue_creator
          -> cue_builder
            -> cue_builder_root
              -> cue_builder_client
                -> class CueBuilderAppRenderer extends AppRenderer
  ```
    - Talk with Robyn regarding when we about this specific call site and approaches we want to take
 - Create a helper method `isFileProviderAppApproved` which takes in a `fileProviderId` and returns the blocking status of that App
 - Update the following call sites to use the helper method above
  - `attachment_helpers.allThirdPartyFileAttachmentTypesDisabled`
  - `third_party_thumbnail._integrationIsEnabled`
  - `services.integrations.isEnabledNonReactive(IntegrationId` (mostly in add_attachment_button.ts)
 - Testing with the feature flag on and off
