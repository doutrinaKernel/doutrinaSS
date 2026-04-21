        let rawEvents = [];  

        lines.forEach(line => {
            const tsMatch = line.match(REGEX.TS_IOS);
            if(!tsMatch) return;

            if(line.includes('ProductVersion:')) hardware.ios = line.split(':')[1].trim();
            if(line.includes('ProductBuildVersion:')) hardware.build = line.split(':')[1].trim();

            const idMatch = line.match(/\b([a-f0-9]{40}|[a-f0-9]{24}|\d{15})\b/i);
            if(idMatch) hardware.ids.add(idMatch[0]);
        });

        document.getElementById('hw-ios').innerText = hardware.ios;
        document.getElementById('hw-build').innerText = hardware.build;
        document.getElementById('hw-id').innerText = Array.from(hardware.ids).join(' | ') || "Not Found";
    }
</script>

</body>
</html>
