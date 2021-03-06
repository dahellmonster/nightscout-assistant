<template>
  <section id="nightscout-settings" class="mt-8 mx-10">
    <form @submit="submit">
      <div class="columns">
        <!-- Nightscout url input -->
        <div class="column is-three-fifths">
          <label class="label">{{ $t("index.settings.your-site") }}</label>
          <div class="control">
            <input
              class="input"
              :class="{ 'is-danger': !nsUrlIsValid }"
              type="text"
              id="nightscout-url"
              v-model="user.settings.nsUrl"
              placeholder="https://MY-NS-SITE.herokuapp.com"
            />
          </div>
          <p v-if="!nsUrlIsValid" class="help is-danger">
            {{ $t("index.settings.invalid-url") }}
          </p>
        </div>

        <!-- Unit input -->
        <div class="column">
          <label class="label">{{ $t("index.settings.unit") }}</label>
          <div class="control">
            <div class="select is-fullwidth">
              <select
                required
                id="nightscout-unit"
                v-model="user.settings.unit"
              >
                <option value="mg/dl">{{ $t("index.settings.mg-dl") }}</option>
                <option value="mmol/l">
                  {{ $t("index.settings.mmol-l") }}
                </option>
              </select>
            </div>
          </div>
        </div>

        <!-- API secret input -->
        <div class="column">
          <label class="label">
            {{ $t("index.settings.api-secret") }}
            <span class="text-xs has-text-grey-light">
              {{ $t("index.settings.optional") }}
            </span>
          </label>
          <div class="control">
            <input
              :placeholder="$t('index.settings.unchanged')"
              v-model="user.settings.apiSecret"
              @focus="showApiSecretInfo = true"
              class="input"
              type="password"
              id="nightscout-api-secret"
            />
          </div>
        </div>
      </div>

      <article class="message" v-show="showApiSecretInfo" id="api-secret-info">
        <div class="message-header">
          <p class="text-sm">
            {{ $t("index.settings.api-secret-info.title") }}
          </p>
          <button
            type="button"
            class="delete"
            aria-label="delete"
            @click="showApiSecretInfo = false"
          ></button>
        </div>
        <div class="message-body">
          <p class="text-sm">
            <i18next path="index.settings.api-secret-info.info">
              <span class="is-family-code has-text-info"
                >AUTH_DEFAULT_ROLES</span
              >
              <span class="is-family-code has-text-info">denied</span>
            </i18next>
          </p>
        </div>
      </article>
      <button class="button is-primary" type="submit">
        {{ $t("index.settings.save") }}
      </button>
      <button class="button is-secondary ml-3" @click="signOut">
        {{ $t("index.settings.sign-out") }}
      </button>
    </form>
    <ns-modal-saved
      v-if="saveSuccess"
      @closed="saveSuccess = false"
      :nsUrl="user.settings.nsUrl"
    ></ns-modal-saved>
  </section>
</template>

<script>
import nsModalSaved from "@/components/ns-modal-saved";
export default {
  name: "ns-user-settings",
  components: {
    nsModalSaved
  },
  computed: {
    nsUrlIsValid() {
      if (!this.user.settings.nsUrl) return true;
      var rex = RegExp(require("@/assets/url-regex.js"));
      var match = this.user.settings.nsUrl.match(rex);
      return match && match.length > 0;
    }
  },
  data() {
    return {
      saveSuccess: false,
      showApiSecretInfo: false,
      user: {
        settings: {
          nsUrl: undefined,
          unit: undefined,
          apiSecret: undefined
        }
      }
    };
  },
  async mounted() {
    let snapshot = await firebase
      .firestore()
      .collection("users")
      .doc(this.$store.state.user.email)
      .get();
    if (snapshot.exists) {
      var data = snapshot.data();
      this.user.settings.nsUrl = data.nsUrl;
      this.user.settings.unit = data.unit;
      this.$store.commit("setUserSettings", this.user.settings);
    }
  },
  methods: {
    signOut() {
      firebase
        .auth()
        .signOut()
        .then(function() {
          window.location.reload();
        });
    },
    submit(e) {
      e.preventDefault();
      let settings = {
        nsUrl: this.user.settings.nsUrl,
        unit: this.user.settings.unit
      };
      if (
        this.user.settings.apiSecret != null &&
        this.user.settings.apiSecret.length > 0
      ) {
        var secretHash = new jsSHA("SHA-1", "TEXT");
        secretHash.update(this.user.settings.apiSecret);
        settings.secretHash = secretHash.getHash("HEX");
      }

      firebase
        .firestore()
        .collection("users")
        .doc(this.$store.state.user.email)
        .set(settings, { merge: true })
        .then(() => {
          this.saveSuccess = true;
        });
    }
  }
};
</script>

<style></style>
