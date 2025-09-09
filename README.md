<img>
import React, { useEffect, useState } from "react";
import { Line } from "react-chartjs-2";
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
} from "chart.js";

ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend);

export default function GithubActivityGraph({ username }) {
  const [data, setData] = useState(null);

  useEffect(() => {
    async function fetchData() {
      const res = await fetch(`https://api.github.com/users/${R1yAA}/events/public`);
      const events = await res.json();

      // Count commits, PRs, issues by day
      const stats = {};
      events.forEach((e) => {
        const day = e.created_at.slice(0, 10);
        stats[day] = stats[day] || { commits: 0, prs: 0, issues: 0 };

        if (e.type === "PushEvent") stats[day].commits += e.payload.commits.length;
        if (e.type === "PullRequestEvent") stats[day].prs += 1;
        if (e.type === "IssuesEvent") stats[day].issues += 1;
      });

      const labels = Object.keys(stats).sort();
      setData({
        labels,
        datasets: [
          { label: "Commits", data: labels.map((d) => stats[d].commits), borderColor: "blue" },
          { label: "PRs", data: labels.map((d) => stats[d].prs), borderColor: "green" },
          { label: "Issues", data: labels.map((d) => stats[d].issues), borderColor: "red" },
        ],
      });
    }
    fetchData();
  }, [username]);

  if (!data) return <p>Loading...</p>;

  return (
    <div className="p-4 bg-gray-900 rounded-xl shadow-lg">
      <Line data={data} options={{ responsive: true, plugins: { legend: { position: "top" } } }} />
    </div>
  );
}

</img>
