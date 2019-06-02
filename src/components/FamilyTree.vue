<template lang="pug">
	#family-tree
</template>

<script>
	import Sigma from 'sigma'
	
	let familyTree
		
	const getName = person => {
		if(typeof person === 'object') {
			const { first, middle, last } = person
			return `${first} ${last}`
		} else {
			return person
		}
	}
	
	let countAncestors 
	countAncestors = (family, member) => {
		const [child, parents = []] = family.find(([child]) => getName(child) === getName(member))
		
		const counts = parents.map(parent => countAncestors(family, parent))
		const greatest = counts.reduce((greatest, count) => count > greatest ? count : greatest, 0)
		
		return 1 + greatest
	}
	
	const countGeneration = (family, generation) =>	family
		.map(([member]) => countAncestors(family, member))
		.filter(count => count === generation)
		.length
	
	const createFamilyTree = (tree, familyData) => {
		const { graph } = tree
		const addedFamily = []
	
		familyData.forEach(([child, parents]) => {
			const generation = countAncestors(familyData, child)
		
			graph.addNode({
				id: getName(child),
				label: getName(child),
				x: countGeneration(addedFamily, generation) / 4,
				y: generation / 4,
				size: 1,
				color: '#f00'
			})
			
			addedFamily.push([child, parents])
			
			if(parents) {
				parents.forEach(parent => {
					graph.addEdge({
						id: `${getName(parent)} - ${getName(child)}`,
						source: getName(parent),
						target: getName(child)
					})
				})
			}
		})
		
		tree.refresh()
	}

	export default {
		mounted() {
			familyTree = new Sigma('family-tree')
			
			createFamilyTree(familyTree, this.family)
		},
		data() {
			return {
				family: [
					['Muhamed'],
					['Redzima'],
					['Todor'],
					['Pera'],
					['Besim', ['Muhamed', 'Redzima']],
					['Cedna', ['Todor', 'Pera']],
					[{
						first: 'Dennis',
						middle: '',
						last: 'Kuduzovic'
					}, ['Besim', 'Cedna']],
					['Nina', ['Besim', 'Cedna']]
				]
			}
		}
	}
</script>

<style scoped>
	#family-tree {
		position: absolute;
		height: 100%;
		width: 100%;
	}
</style>