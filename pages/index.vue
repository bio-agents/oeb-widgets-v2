<template>
  <v-row>
    <v-col id="app" data-app>
      <LoaderWidgets v-if="this.fetchedData" :dataChart="this.preparedData"></LoaderWidgets>
    </v-col>
  </v-row>
</template>
<script>
export default {
  name: 'IndexPage',
  data() {
    return {
      fetchedData: null,
    }
  },
  async mounted() {
    // Fetch your data
    const response = await fetch('./OEBD01000000QR.json');
    this.fetchedData = await response.json();
    this.fetchDataAndRender(this.fetchedData)

  },
  methods: {
    async fetchDataAndRender(data) {
      // Sets charging status based on data presence
      this.loading = !data;
      let visualization = data.inline_data.visualization
      let type = visualization.type
      // Prepare the data to pass to the component
      this.preparedData = {
        _id: data._id,
        dates: data.dates,
        dataset_contact_ids: data.dataset_contact_ids,
        inline_data: {
          challenge_participants:[],
          visualization:{}
        }
      }
      
      // Prepare specific data for Plots
      if (type === 'bar-plot'){
        // Process challenge_participants data for BarPlot
        data.inline_data.challenge_participants.forEach(participant => {
          const preparedParticipant = {
            agent_id: participant.agent_id,
            metric_value: participant.metric_value,
            stderr: participant.stderr
          };
          this.preparedData.inline_data.challenge_participants.push(preparedParticipant);
        });
        // Process visualization data for BarPlot
        const visualization = data.inline_data.visualization;
        this.preparedData.inline_data.visualization = {
          metric: visualization.metric,
          type: visualization.type
        };
      }else if (type === '2D-plot'){
        // Process challenge_participants data for ScatterPlot
        data.inline_data.challenge_participants.forEach(participant => {
          const preparedParticipant = {
            agent_id: participant.agent_id,
            metric_x: participant.metric_x,
            stderr_x: participant.stderr_x,
            metric_y: participant.metric_y,
            stderr_y: participant.stderr_y
          };
          this.preparedData.inline_data.challenge_participants.push(preparedParticipant);
        });
        // Process visualization data for ScatterPlot
        const visualization = data.inline_data.visualization;
        // const metrics_names = await this.getMetricsNames(visualization.x_axis, visualization.y_axis);
        this.preparedData.inline_data.visualization = {
          type: visualization.type,
          x_axis: visualization.x_axis,
          y_axis: visualization.y_axis,
          optimization: visualization.optimization
        };
      } else if (type === 'box-plot'){
        // Process challenge_participants data for ScatterPlot
        data.inline_data.challenge_participants.forEach(participant => {
          const preparedParticipant = {
            name: participant.name,
            metric_id: participant.metric_id,
            q1: participant.q1,
            mean: participant.mean,
            median: participant.median,
            q3: participant.q3,
            lowerfence: participant.lowerfence,
            upperfence: participant.upperfence
          };
          this.preparedData.inline_data.challenge_participants.push(preparedParticipant);
        });
        // Process visualization data for ScatterPlot
        const visualization = data.inline_data.visualization;
        // const metrics_names = await this.getMetricsNames(visualization.x_axis, visualization.y_axis);
        this.preparedData.inline_data.visualization = {
          type: visualization.type,
          y_axis: visualization.y_axis,
          optimization: visualization.optimization?visualization.optimization:null
        };
      }else{
        return null;
      }
      
      // Check the display type and set the corresponding status
      if (visualization && type) {
        this.visualizationType = type;
      }
    },
  },
}
</script>
