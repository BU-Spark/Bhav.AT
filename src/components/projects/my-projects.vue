<template>
  <main>
    <section class="jumbotron text-center">
      <div class="container">
        <h1 class="jumbotron-heading">My Projects</h1>
        <p>
          <router-link :to="{ name: 'createproject' }">
            <button class="btn btn-primary">Create Project</button>
          </router-link>
        </p>
      </div>
    </section>
    <div class="container">
      <masonry
        :cols="{ default: 3, 991: 2, 767: 1 }"
        :gutter="{ default: '30px', 767: '15px' }"
      >
        <div
          v-for="project in projects"
          :key="project.id"
          class="card mb-4 box-shadow"
        >
          <div class="card-header">
            <h2 class="my-0 font-weight-normal">{{ project.data().name }}</h2>
          </div>
          <div class="card-body">
            <h4 class="card-title">Short Introduction</h4>
            <p>{{ project.data().intro }}</p>
            <h4>Members</h4>
            <p v-for="member in project.data().members" :key="member.id">
              <router-link :to="{ name: 'profile', params: { uname: member } }"
                ><span>{{ member }}</span></router-link
              >
            </p>
            <router-link
              :to="{ name: 'project', params: { name: project.data().name } }"
            >
              <button
                v-if="project.data().members.find(item => item == user.uname)"
                class="btn btn-lg btn-block btn-primary"
              >
                View
              </button>
            </router-link>
          </div>
        </div>
      </masonry>
    </div>
    <hr class="featurette-divider" />

    <footer class="container">
      <p class="float-right"><a href="#">Back to top</a></p>
      <p>
        © 2020 BU Spark! ·
        <a>
          <router-link :to="{ name: 'privacy' }">Privacy</router-link>
        </a>
      </p>
    </footer>
  </main>
</template>

<script>
import { db } from "@/firebase/init.js";
export default {
  name: "myprojects",
  computed: {
    user() {
      return this.$store.state.user;
    }
  },
  data() {
    return {
      projects: null
    };
  },
  async created() {
    let projectsGet = await db
      .collection("projects")
      .where("members", "array-contains", this.user.uname)
      .get();
    this.projects = projectsGet.docs;
  }
};
</script>

<style>
.container {
  padding-top: 40px;
  padding-bottom: 40px;
}

.material-icons.green {
  color: green;
}

.material-icons.red {
  color: red;
}

.availability {
  padding-top: 6px;
  padding-left: 3px;
}

.section-head {
  padding-left: 8px;
}

.content {
  padding-left: 12px;
}

.green {
  color: green;
}

.red {
  color: red;
}
</style>
